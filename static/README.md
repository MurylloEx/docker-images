# Static (HTML / CSS / JS)

Multi-stage image with **Nginx Alpine** for static sites (HTML, CSS, and plain JavaScript, no bundler required).

## Project requirements

Place static files in this directory root, for example:

- `index.html`
- `css/`, `js/`, `assets/`

Nginx uses `try_files` with a fallback to `index.html` (useful for simple SPAs).

## Local usage

```bash
cp .env.example .env
docker compose up --build
```

Access at `http://localhost:${HOST_PORT}` (default `8000`).

`docker-compose` mounts the current directory to `/usr/share/nginx/html` (read-only) for development without rebuilds.

## Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `APP_PORT` | `8000` | Nginx port in the container (build arg) |
| `HOST_PORT` | `8000` | Host port |
| `APP_UID` | `1000` | Nginx user UID |
| `APP_USER` | `nginx` | Username |

## Nginx configuration

File: [nginx/default.conf](nginx/default.conf). The `__APP_PORT__` placeholder is replaced at build time.

## Manual build

```bash
docker compose build
docker compose up
```

## CI

Use `STACK=static` when triggering Jenkins. See [root README](../README.md#cicd).
