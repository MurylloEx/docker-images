# Rust

Multi-stage image for Rust applications: compile on `rust:1.85-alpine` and run on a minimal `alpine:3.21` runtime.

## Project requirements

Place the following in this directory root:

- `Cargo.toml`
- `Cargo.lock` (recommended)
- `src/` with application code

The binary must listen on the port defined by `APP_PORT` (runtime environment variable).

## Local usage

```bash
cp .env.example .env
docker compose up --build
```

Access at `http://localhost:${HOST_PORT}` (default `8000`).

## Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `APP_PORT` | `8000` | Process port inside the container |
| `HOST_PORT` | `8000` | Mapped host port |
| `APP_UID` | `1000` | `app` user UID |
| `APP_USER` | `app` | Username |
| `APP_BIN` | (empty) | Cargo binary name; empty = auto-detect |

## Manual build

```bash
docker compose build
docker compose run --rm app
```

## CI

Use `STACK=rust` when triggering Jenkins. See [root README](../README.md#cicd).
