# Rust

Multi-stage image for Rust: compile on `rust:1.85-alpine`, run a single binary on `alpine:3.21`.

## What ships here

- `Dockerfile`, `docker-compose.yml`, `.env.example`
- No application code (add your Rust project in this directory)

## Project requirements

| File / folder | Required |
|---------------|----------|
| `Cargo.toml` | Yes |
| `Cargo.lock` | Recommended |
| `src/` | Yes |

The binary must listen on `0.0.0.0` using the `APP_PORT` environment variable at runtime.

## Local usage

```bash
cp .env.example .env
docker compose up --build
```

URL: `http://localhost:${HOST_PORT}` (default `8000`).

### Development volumes

Compose mounts `.` → `/app`. That **replaces** the compiled `/app/bin` from the image. For a production-like test, run without the bind mount or build on the host:

```bash
docker compose build
docker run --rm -p 8000:8000 -e APP_PORT=8000 $(docker compose images -q app)
```

## Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `APP_PORT` | `8000` | Process port in the container |
| `HOST_PORT` | `8000` | Host port |
| `APP_UID` | `1000` | `app` user UID |
| `APP_USER` | `app` | Username |
| `APP_BIN` | (empty) | Cargo binary name; empty = auto-detect via `find` |

## Build details

- Dependency cache layer uses a dummy `src/main.rs`, then the real sources are copied.
- Release binary is copied to `/app/bin` in the runtime image.

## Commands

```bash
docker compose build
docker compose run --rm app
```

## CI

Use `STACK=rust` when triggering Jenkins. See [root README](../README.md#cicd-jenkins).
