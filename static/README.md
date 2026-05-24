# Static (Nginx)

Multi-stage image with **Nginx Alpine** for static sites and SPAs (HTML, CSS, JS). Supports plain files in the build context or a pre-built frontend `dist/`.

## What ships here

- `Dockerfile`, `docker-compose.yml`, `.env.example`, [nginx/default.conf](nginx/default.conf)
- No site content by default (add files before build)

## Project requirements

### Option A — plain static files

Place assets in this directory root (sibling to `Dockerfile`):

```text
static/
├── index.html
├── css/
├── js/
└── assets/
```

Docker build copies the context (excluding Docker metadata) into `/usr/share/nginx/html`.

### Option B — SPA build output

Build your frontend locally, then copy output here before `docker compose build`:

```bash
# example: Vite/React in another folder
npm run build
cp -r ../my-app/dist/* .
docker compose up --build
```

Or keep sources in a subfolder (e.g. `smartru-front/`) and copy only `dist/` into the static root for the image build.

Nginx uses `try_files` with fallback to `index.html` for client-side routing.

## Local usage

```bash
cp .env.example .env
docker compose up --build
```

URL: `http://localhost:${HOST_PORT}` (default `8000`).

### Development volumes

Compose mounts `.` → `/usr/share/nginx/html` (read-only). Whatever is on the host is what Nginx serves—useful for iterating without rebuilds.

## Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `APP_PORT` | `8000` | Nginx listen port (build arg) |
| `HOST_PORT` | `8000` | Host port |
| `APP_UID` | `1000` | Nginx user UID |
| `APP_USER` | `nginx` | Username |

## Nginx

[nginx/default.conf](nginx/default.conf) — `__APP_PORT__` is substituted at build time. Nginx runs as non-root (`pid` under `/tmp`, `user` directive removed from main config).

## Commands

```bash
docker compose build
docker compose up
```

## CI

Use `STACK=static` when triggering Jenkins. See [root README](../README.md#cicd-jenkins).
