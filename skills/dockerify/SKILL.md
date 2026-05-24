---
name: dockerify
description: >-
  Dockerize applications using the docker-images stack templates (rust, python,
  nestjs, static, php): copy stack plumbing, wire .env and compose, validate app
  requirements, and integrate Jenkins remote triggers (GIT_BRANCH, STACK).
  Use when dockerizing a repo, adding Docker to a project, containerizing for
  Jenkins, or matching stacks from the docker-images repository.
---

# Dockerify

Turn an application into a container using the **stack templates** in this repository (`rust/`, `python/`, `nestjs/`, `static/`, `php/`). Builds are intended to run on **Jenkins** (Docker agent); Git hosts only trigger the job via [jenkins/](../../jenkins/).

## Quick decision

| Project signals | Stack | Template dir |
|-----------------|-------|----------------|
| `Cargo.toml`, `src/main.rs` | `rust` | [rust/](../../rust/) |
| ASGI/FastAPI, `requirements.txt`, `app/` | `python` | [python/](../../python/) |
| NestJS, `package.json`, `nest build` → `dist/` | `nestjs` | [nestjs/](../../nestjs/) |
| SPA/static, `dist/` or `index.html` | `static` | [static/](../../static/) |
| PHP, `public/index.php` or Laravel/Symfony | `php` | [php/](../../php/) |

If ambiguous, ask once. Do not invent a custom Dockerfile when a stack fits.

## Workflow checklist

Copy and track progress:

```
Dockerify progress:
- [ ] 1. Pick stack
- [ ] 2. Lay out app + stack files
- [ ] 3. Copy template plumbing from docker-images
- [ ] 4. Configure .env (from .env.example)
- [ ] 5. Satisfy stack requirements (listen on APP_PORT)
- [ ] 6. Local build: docker compose up --build
- [ ] 7. Jenkins: Jenkinsfile + remote trigger (optional)
```

## Step 1 — Pick stack

Use the table above. One stack per deployable service.

## Step 2 — Layout (choose one)

**A — App lives inside stack folder (recommended for this repo)**

```text
my-app-repo/
└── rust/                 # or python/, nestjs/, static/, php/
    ├── Dockerfile        # from template
    ├── docker-compose.yml
    ├── .env.example
    ├── .env              # gitignored
    ├── Cargo.toml        # app files at same level as Dockerfile
    └── src/
```

**B — App at repo root, stack in subfolder**

Keep application at root only if the Dockerfile build context can `COPY` it; prefer **A** to match templates unchanged.

**C — Frontend + API**

Use two folders: e.g. `static/` for SPA build output, `python/` or `nestjs/` for API—two images, two Jenkins `STACK` values or two jobs.

## Step 3 — Copy template plumbing

From **this** repository, copy into the target project (do not copy `.agents/` or other stacks):

| Copy | Notes |
|------|--------|
| `{stack}/Dockerfile` | As-is unless app needs `APP_BIN`, `WEB_ROOT`, `APP_MODULE` |
| `{stack}/docker-compose.yml` | As-is |
| `{stack}/.env.example` | → `.env` in step 4 |
| `{stack}/.dockerignore` | Merge with project ignores |
| `{stack}/nginx/` or `{stack}/php-fpm/` | Only for `static` / `php` |

Do **not** strip multi-stage stages or non-root user setup unless the user explicitly requests it.

Stack-specific build args (see [stacks.md](stacks.md)):

- `rust`: `APP_BIN` if binary name ≠ crate name
- `python`: `APP_MODULE` (default `app.main:app`)
- `php`: `WEB_ROOT` (`public` or `.`)
- `static`: put built assets in context root before build (or copy `dist/` in CI)

## Step 4 — Environment

```bash
cp .env.example .env
```

Shared defaults (all stacks):

| Variable | Default | Notes |
|----------|---------|--------|
| `APP_PORT` | `8000` | App/listen port in container; rebuild if changed |
| `HOST_PORT` | `8000` | Published host port |
| `APP_UID` | `1000` | Non-root user |
| `APP_USER` | stack-specific | See stack README |

## Step 5 — Application requirements (must verify)

Before claiming success, confirm:

| Stack | Must |
|-------|------|
| **rust** | Listen on `0.0.0.0` using `APP_PORT` env |
| **python** | ASGI app at `APP_MODULE`; Uvicorn CMD in image |
| **nestjs** | `listen` uses `process.env.PORT ?? process.env.APP_PORT ?? 8000` |
| **static** | `index.html` (and `dist/` assets) in build context |
| **php** | Entry under `WEB_ROOT` (e.g. `public/index.php`) |

Read the stack README under `{stack}/README.md` for details.

## Step 6 — Local validation

```bash
cd <stack-folder>
docker compose build
docker compose up
# curl http://localhost:${HOST_PORT:-8000}
```

**Bind mounts:** compose files mount the project dir for dev. That can hide the image-built binary (`rust`) or require host `dist/` (`nestjs`). For production-like test:

```bash
docker compose build
docker run --rm -p 8000:8000 --env-file .env $(docker compose images -q app)
```

## Step 7 — Jenkins integration

Git hosting **does not build images**—it triggers Jenkins with `GIT_BRANCH` (and optionally `STACK`).

### 7a — Remote trigger (GitHub / Bitbucket / GitLab)

Copy from [jenkins/](../../jenkins/) into the **application** repository:

| Platform | Source | Destination |
|----------|--------|-------------|
| GitHub | `jenkins/github/jenkins.yml` | `.github/workflows/jenkins.yml` |
| Bitbucket | `jenkins/bitbucket/bitbucket-pipelines.yml` | `bitbucket-pipelines.yml` |
| GitLab | `jenkins/gitlab/gitlab-ci.yml` | `.gitlab-ci.yml` |

Configure secrets/variables per [jenkins/README.md](../../jenkins/README.md) (`JENKINS_URL`, `JENKINS_USER`, `JENKINS_API_TOKEN`, `JENKINS_BUILD_TOKEN`, `JENKINS_JOB_NAME`).

Extend the trigger to pass **`STACK`** if the Jenkins job supports it (see Jenkinsfile below). Example query param on `buildWithParameters`:

```text
STACK=rust
```

### 7b — Jenkins job (build on agent)

Install [jenkins/Jenkinsfile](../../jenkins/Jenkinsfile) in Jenkins (Pipeline from SCM or copy into app repo root).

Parameters:

| Parameter | Purpose |
|-----------|---------|
| `GIT_BRANCH` | Branch to build (from CI trigger) |
| `STACK` | `rust` \| `python` \| `nestjs` \| `static` \| `php` |
| `IMAGE_NAME` | Optional image name (default `app`) |
| `IMAGE_TAG` | Optional tag (default `GIT_COMMIT_SHORT` or `latest`) |

Requirements:

- Agent label `docker` (or change `agent` in Jenkinsfile)
- Docker CLI + socket on the agent
- Checkout of the repo at `GIT_BRANCH`
- For `static` SPA: CI stage may need `npm run build` before `docker compose build` if sources are not pre-built

Full pipeline details: [jenkins.md](jenkins.md).

## What to deliver

When dockerifying for the user, produce:

1. Chosen stack and layout (A/B/C)
2. List of files added/updated (Dockerfile, compose, `.env.example`, `.dockerignore`, nginx/php-fpm if any)
3. App code changes (port binding, module path, `WEB_ROOT`, etc.)
4. Jenkins: trigger workflow + note to configure secrets; Jenkinsfile parameters `GIT_BRANCH` + `STACK`
5. Commands to build and run locally

## Anti-patterns

- Do not use `latest`-only single-stage images when a stack template exists
- Do not run production builds only on GitHub Actions runners if the project standard is Jenkins
- Do not change `APP_PORT` in `.env` without noting that Nginx/static/php need **image rebuild**
- Do not commit `.env` (secrets)

## Additional resources

- Per-stack variables and pitfalls: [stacks.md](stacks.md)
- Jenkins pipeline and trigger extensions: [jenkins.md](jenkins.md)
