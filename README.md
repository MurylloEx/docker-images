# docker-images

Multi-stage Docker images (Alpine) ready to host applications, with `docker-compose`, `.env.example`, and CI templates that trigger builds on an **external Jenkins** instance.

## Stacks

| Stack | Directory | Description |
|-------|-----------|-------------|
| Rust | [rust/](rust/) | Release build on `rust:alpine`, runtime on `alpine` |
| Python | [python/](python/) | ASGI with Uvicorn (`alpine:3.21` + Python 3 via `apk`) |
| NestJS | [nestjs/](nestjs/) | Node LTS Alpine, `dist/` build |
| Static | [static/](static/) | HTML/CSS/JS served by Nginx |
| PHP Laravel | [php-laravel/](php-laravel/) | Nginx + PHP-FPM via Unix socket |

## Quick start

```bash
cd rust
cp .env.example .env
docker compose up --build
```

The default port is **8000** inside the container (`APP_PORT`). On the host, set `HOST_PORT` in `.env`.

## Common variables

| Variable | Default | Purpose |
|----------|---------|---------|
| `APP_PORT` | `8000` | Container internal port (build + runtime) |
| `HOST_PORT` | `8000` | Published host port |
| `APP_UID` | `1000` | Non-root user UID |
| `APP_USER` | (per stack) | Container username |

Changing `APP_PORT` requires an image **rebuild** (Nginx and other configs use build args).

## CI/CD

Image builds run on **Jenkins**. GitHub Actions and Bitbucket only **trigger** the remote job.

| Template | Path |
|----------|------|
| Jenkinsfile | [templates/Jenkinsfile](templates/Jenkinsfile) |
| GitHub Actions | [templates/github-actions/trigger-jenkins.yml](templates/github-actions/trigger-jenkins.yml) |
| Bitbucket Pipelines | [templates/bitbucket-pipelines.yml](templates/bitbucket-pipelines.yml) |

### Installing the templates

| Platform | Source | Destination in your repository |
|----------|--------|--------------------------------|
| GitHub Actions | `templates/github-actions/trigger-jenkins.yml` | `.github/workflows/trigger-jenkins.yml` |
| Bitbucket | `templates/bitbucket-pipelines.yml` | `bitbucket-pipelines.yml` (root) |
| Jenkins | `templates/Jenkinsfile` | **Pipeline script from SCM** job |

On Jenkins, enable **Trigger builds remotely** or use basic authentication with an API token on `buildWithParameters`. Job parameters must match those defined in the Jenkinsfile (`STACK`, `GIT_COMMIT`, etc.).

### Jenkins setup

1. Create a **Pipeline** job with **Pipeline script from SCM**, script path `templates/Jenkinsfile`.
2. Parameters are defined via `properties([parameters([...])])` in the Jenkinsfile (the first build may ignore them; they appear from the second build onward).
3. Install plugins: Pipeline, Git, Docker (agent with Docker socket).
4. Agent with label `docker` (adjust in the Jenkinsfile if needed).
5. For registry push, configure the `docker-registry` credential (Username with password).

### Secrets / variables on Git providers

**GitHub** (Settings → Secrets and variables):

| Secret / Variable | Description |
|-------------------|-------------|
| `JENKINS_URL` | Base URL, e.g. `https://jenkins.example.com` |
| `JENKINS_USER` | User with build permission |
| `JENKINS_API_TOKEN` | User API token |
| `JENKINS_JOB_NAME` (variable) | Job name, e.g. `docker-images-build` |
| `JENKINS_BUILD_TOKEN` (secret, optional) | Job `token` parameter |

**Bitbucket** (Repository variables, secured):

| Variable | Description |
|----------|-------------|
| `JENKINS_URL` | Jenkins base URL |
| `JENKINS_USER` | Username |
| `JENKINS_API_TOKEN` | API token |
| `JENKINS_JOB_NAME` | Job name |
| `JENKINS_BUILD_TOKEN` | Optional job token |

### Parameters sent to Jenkins

| Parameter | Description |
|-----------|-------------|
| `STACK` | `rust`, `python`, `nestjs`, `static`, `php-laravel` |
| `GIT_COMMIT` | Source commit SHA |
| `GIT_BRANCH` | Source branch |
| `SOURCE_REPO` | Source repository |
| `IMAGE_TAG` | Image tag (optional) |

## Repository structure

```
.
├── rust/
├── python/
├── nestjs/
├── static/
├── php-laravel/
└── templates/
    ├── Jenkinsfile
    ├── bitbucket-pipelines.yml
    └── github-actions/
        └── trigger-jenkins.yml
```

## License

See [LICENSE](LICENSE).
