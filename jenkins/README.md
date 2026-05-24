# Jenkins remote trigger

CI workflows that **only trigger** a Jenkins job (`buildWithParameters`). The Docker build runs on Jenkins, not on GitHub / Bitbucket / GitLab runners.

Default Jenkins job name: `smartru` (override with `JENKINS_JOB_NAME` / `vars.JENKINS_JOB_NAME`).

Parameters sent by default: `GIT_BRANCH` (plus `token` when using a build token).

Add `STACK` (`rust`, `python`, `nestjs`, `static`, `php`) to `buildWithParameters` for multi-stack repos — see [Jenkinsfile](Jenkinsfile) and the [dockerify](../skills/dockerify/SKILL.md) skill.

## Files

| Platform | Template | Install to |
|----------|----------|------------|
| GitHub Actions | [github/jenkins.yml](github/jenkins.yml) | `.github/workflows/jenkins.yml` |
| Bitbucket Pipelines | [bitbucket/bitbucket-pipelines.yml](bitbucket/bitbucket-pipelines.yml) | `bitbucket-pipelines.yml` (repository root) |
| GitLab CI | [gitlab/gitlab-ci.yml](gitlab/gitlab-ci.yml) | `.gitlab-ci.yml` (repository root) |

## Triggers

| Platform | Automatic | Manual |
|----------|-----------|--------|
| GitHub | Push to `main`, `dev` | Actions → **jenkins** → Run workflow (`git_branch` input) |
| Bitbucket | Push to `main`, `dev` | Pipelines → **Run pipeline** → custom **jenkins** |
| GitLab | Push to `main`, `dev` | CI/CD → **Run pipeline** (web source) |

## Jenkins

1. Create a **Pipeline** job from [Jenkinsfile](Jenkinsfile) (SCM or copied into the app repo).
2. Job parameters: `GIT_BRANCH`, `STACK` (choice), optional `IMAGE_NAME` / `IMAGE_TAG`, and optionally `token` (remote trigger).
3. Enable **Prevent Cross Site Request Forgery exploits** if you use the crumb flow (templates fetch `crumbIssuer` before POST).
4. Agent with label `docker` and Docker socket for `docker compose build` inside `{STACK}/`.

Point the job at the repository that contains stack folders (`rust/`, `python/`, etc.) and pass `GIT_BRANCH` + `STACK` on each build.

## Secrets and variables

### GitHub (Settings → Secrets and variables → Actions)

| Name | Type | Description |
|------|------|-------------|
| `JENKINS_URL` | Variable | Base URL, e.g. `https://jenkins.example.com` |
| `JENKINS_JOB_NAME` | Variable | Job name (default in script: `smartru`) |
| `STACK` | Variable | `rust`, `python`, `nestjs`, `static`, or `php` (default `rust`) |
| `JENKINS_USER` | Secret | Jenkins user |
| `JENKINS_API_TOKEN` | Secret | API token |
| `JENKINS_BUILD_TOKEN` | Secret | Remote build token (if configured on the job) |

### Bitbucket (Repository variables)

| Name | Secured | Description |
|------|---------|-------------|
| `JENKINS_URL` | No | Base URL |
| `JENKINS_JOB_NAME` | No | Job name |
| `JENKINS_USER` | Yes | Username |
| `JENKINS_API_TOKEN` | Yes | API token |
| `JENKINS_BUILD_TOKEN` | Yes | Build token |

### GitLab (Settings → CI/CD → Variables)

| Name | Protected / masked | Description |
|------|-------------------|-------------|
| `JENKINS_URL` | Recommended | Base URL |
| `JENKINS_JOB_NAME` | No | Job name |
| `JENKINS_USER` | Masked | Username |
| `JENKINS_API_TOKEN` | Masked | API token |
| `JENKINS_BUILD_TOKEN` | Masked | Build token |

Optional manual override: set `GIT_BRANCH` when running the pipeline (Bitbucket custom **jenkins**, GitLab **Run pipeline** variables).

## CSRF crumb

All three templates call `/crumbIssuer/api/json` and send the crumb header on `buildWithParameters`. If CSRF is disabled on Jenkins, you can remove the crumb lines and keep a plain authenticated POST.

## Concurrency (GitHub only)

The GitHub workflow uses `concurrency: jenkins-${{ github.ref }}` with `cancel-in-progress: true` so only the latest push per branch triggers a run.
