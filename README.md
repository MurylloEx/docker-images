# docker-images

[![skills.sh](https://skills.sh/b/MurylloEx/docker-images)](https://skills.sh/MurylloEx/docker-images)

Multi-stage Docker images (Alpine) ready to host applications. Each stack folder ships **only** the image plumbing (`Dockerfile`, `docker-compose.yml`, `.env.example`, and stack docs). **Add your application code** in that folder before building.

Default container port is **8000** (`APP_PORT`). Map another host port with `HOST_PORT` in `.env`.

## Stacks

| Stack | Directory | Description |
|-------|-----------|-------------|
| Rust | [rust/](rust/) | Release build on `rust:1.85-alpine`, minimal `alpine:3.21` runtime |
| Python | [python/](python/) | ASGI with Uvicorn (`alpine:3.21`, Python 3 via `apk`) |
| NestJS | [nestjs/](nestjs/) | Node LTS Alpine, `npm ci` + `npm run build`, production runtime |
| Static | [static/](static/) | Static assets or SPA `dist/` served by Nginx Alpine |
| PHP | [php/](php/) | PHP 8.4 FPM + Nginx over Unix socket (framework-agnostic) |

## Quick start

```bash
cd rust
cp .env.example .env
docker compose up --build
```

Open `http://localhost:${HOST_PORT}` (default `http://localhost:8000`).

## Common variables

| Variable | Default | Purpose |
|----------|---------|---------|
| `APP_PORT` | `8000` | Port inside the container (build arg + runtime) |
| `HOST_PORT` | `8000` | Published port on the host |
| `APP_UID` | `1000` | Non-root user UID |
| `APP_USER` | (per stack) | Container username |

Changing `APP_PORT` requires an image **rebuild** (Nginx and similar configs are applied at build time).

## What each folder contains

| Path | Purpose |
|------|---------|
| `Dockerfile` | Multi-stage image definition |
| `docker-compose.yml` | Local run and port mapping |
| `.env.example` | Copy to `.env` and adjust |
| `README.md` | Stack-specific requirements |

Application source code (`src/`, `package.json`, `Cargo.toml`, etc.) is **not** included—you add it per project.

## CI/CD (Jenkins)

Image builds run on **Jenkins** (agent with Docker). Git hosting only triggers the job.

Copy the template for your platform from [jenkins/](jenkins/):

| Platform | Template |
|----------|----------|
| GitHub Actions | [jenkins/github/jenkins.yml](jenkins/github/jenkins.yml) → `.github/workflows/jenkins.yml` |
| Bitbucket | [jenkins/bitbucket/bitbucket-pipelines.yml](jenkins/bitbucket/bitbucket-pipelines.yml) → `bitbucket-pipelines.yml` |
| GitLab | [jenkins/gitlab/gitlab-ci.yml](jenkins/gitlab/gitlab-ci.yml) → `.gitlab-ci.yml` |

See [jenkins/README.md](jenkins/README.md) for secrets, variables, and triggers.

## Agent skill: dockerify

Use the [dockerify](skills/dockerify/SKILL.md) skill in Cursor to dockerize an application with these stacks and wire Jenkins (`GIT_BRANCH`, `STACK`). It documents layout, env vars, validation, and CI integration.

Install via [skills.sh](https://skills.sh/):

```bash
npx skills add MurylloEx/docker-images --skill dockerify -a cursor -y
```

## Repository structure

```
.
├── jenkins/
│   └── Jenkinsfile
├── skills/
│   └── dockerify/
├── rust/
├── python/
├── nestjs/
├── static/
│   └── nginx/
└── php/
    ├── nginx/
    └── php-fpm/
```

## License

See [LICENSE](LICENSE).
