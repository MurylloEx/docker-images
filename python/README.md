# Python

Multi-stage image for ASGI apps (FastAPI, Starlette, etc.): **Uvicorn** on `alpine:3.21` with Python 3 installed via `apk`.

## What ships here

- `Dockerfile`, `docker-compose.yml`, `.env.example`
- No application code (add your project in this directory)

## Project requirements

| File / folder | Required |
|---------------|----------|
| Application package (e.g. `app/`) | Yes |
| `requirements.txt` | Optional (extra deps; Uvicorn is pre-installed in the image) |

Default command:

```text
uvicorn ${APP_MODULE} --host 0.0.0.0 --port ${APP_PORT}
```

The `python` command is a symlink to `python3`.

## Local usage

```bash
cp .env.example .env
docker compose up --build
```

URL: `http://localhost:${HOST_PORT}` (default `8000`).

### Development volumes

Compose mounts `.` → `/app`, which is suitable for editing code and rebuilding dependencies on the host or via rebuild.

### Example layout

```text
python/
├── app/
│   ├── __init__.py
│   └── main.py      # FastAPI: app = FastAPI()
├── requirements.txt
└── Dockerfile
```

Example `requirements.txt`:

```text
fastapi
```

## Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `APP_PORT` | `8000` | Uvicorn port |
| `HOST_PORT` | `8000` | Host port |
| `APP_UID` | `1000` | User UID |
| `APP_USER` | `app` | Username |
| `APP_MODULE` | `app.main:app` | ASGI import path (`module:variable`) |

## Build details

- Builder installs packages with `pip --prefix=/install`.
- Runtime copies site-packages into `/usr/lib/python3.12/site-packages` (Alpine `sys.path`).

## Commands

```bash
docker compose build
docker compose up
```

## CI

Use `STACK=python` when triggering Jenkins. See [root README](../README.md#cicd-jenkins).
