# Jenkins + Dockerify

## Architecture

```text
Git push (main/dev)
    → GitHub Actions / Bitbucket / GitLab (jenkins templates)
        → POST buildWithParameters (GIT_BRANCH [, STACK])
            → Jenkins Pipeline (Jenkinsfile)
                → checkout → docker compose build in ${STACK}/
```

Image build runs **only on Jenkins**, not on Git host runners.

## Job parameters

| Name | Source | Required |
|------|--------|----------|
| `GIT_BRANCH` | CI trigger | Yes |
| `token` | CI secret | If job uses remote trigger token |
| `STACK` | CI or manual | Yes for multi-stack repos |
| `IMAGE_NAME` | Manual/default | No |
| `IMAGE_TAG` | Manual/default | No |

## Jenkinsfile

Use [jenkins/Jenkinsfile](../../jenkins/Jenkinsfile) from this repo. Create a Pipeline job:

- **Definition:** Pipeline script from SCM, or copy Jenkinsfile into the app repo
- **Parameters:** `GIT_BRANCH`, `STACK` (choice), optional image name/tag
- **Agent:** label `docker`, Docker socket available

## Extending CI triggers with STACK

Default templates only send `GIT_BRANCH`. For monorepos with multiple stacks, add to the POST body (GitHub `run` step example):

```bash
--data-urlencode "STACK=${STACK:-rust}"
```

Set `STACK` from:

- repo variable per project, or
- workflow input on `workflow_dispatch`, or
- path filter (e.g. changes under `python/` → `STACK=python`)

Bitbucket/GitLab: repository variable `STACK` or custom pipeline variable.

## Static frontends on Jenkins

If sources are not committed as built assets:

```groovy
stage('Build SPA') {
  when { expression { params.STACK == 'static' } }
  steps {
    dir('static') {
      sh 'npm ci && npm run build'   // adjust path to frontend root
    }
  }
}
```

Then `docker compose build` in `static/` (context must include `index.html` / assets).

## Secrets checklist

| Variable | Where |
|----------|--------|
| `JENKINS_URL` | CI variables |
| `JENKINS_JOB_NAME` | CI variables |
| `JENKINS_USER` | CI secret |
| `JENKINS_API_TOKEN` | CI secret |
| `JENKINS_BUILD_TOKEN` | CI secret |

See [jenkins/README.md](../../jenkins/README.md).

## Verify after setup

1. `docker compose build` locally in `{STACK}/`
2. Trigger Jenkins manually with `GIT_BRANCH=main`, `STACK=rust` (etc.)
3. Confirm agent runs `docker compose` and image is produced/tag pushed if configured
