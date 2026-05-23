# Python

Multi-stage image for ASGI applications (FastAPI, Starlette, etc.) with **Uvicorn** on `alpine:3.21`, with Python 3 installed via `apk`.

## Project requirements

Place the following in this directory root:

- Application code (e.g. `app/` package)
- `requirements.txt` with project dependencies

Uvicorn is pre-installed in the image. The `python` command points to `python3` (symlink). Default entrypoint:

```text
uvicorn ${APP_MODULE} --host 0.0.0.0 --port ${APP_PORT}
```

## Local usage

```bash
cp .env.example .env
docker compose up --build
```

Access at `http://localhost:${HOST_PORT}` (default `8000`).

## Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `APP_PORT` | `8000` | Uvicorn port |
| `HOST_PORT` | `8000` | Host port |
| `APP_UID` | `1000` | User UID |
| `APP_USER` | `app` | Username |
| `APP_MODULE` | `app.main:app` | ASGI module (`package.module:app`) |

## Example `requirements.txt`

```text
fastapi
sqlalchemy
```

## Manual build

```bash
docker compose build
docker compose up
```

## CI

Use `STACK=python` when triggering Jenkins. See [root README](../README.md#cicd).
