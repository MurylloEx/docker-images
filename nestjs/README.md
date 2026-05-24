# NestJS

Multi-stage image for **NestJS** on `node:lts-alpine`: `npm ci`, `npm run build`, runtime with production dependencies only.

## What ships here

- `Dockerfile`, `docker-compose.yml`, `.env.example`
- No application code (add your NestJS project in this directory)

## Project requirements

Add a standard NestJS app at this directory root:

| File / folder | Required |
|---------------|----------|
| `package.json` | Yes |
| `package-lock.json` | Recommended (`npm ci` in the image) |
| `src/` | Yes |
| `build` script | Must output `dist/` |

In `src/main.ts`, listen on the environment port:

```typescript
await app.listen(process.env.PORT ?? process.env.APP_PORT ?? 8000);
```

Compose sets `PORT` and `APP_PORT` from `.env`.

## Local usage

```bash
cp .env.example .env
docker compose up --build
```

URL: `http://localhost:${HOST_PORT}` (default `8000`).

### Development volumes

Compose mounts:

- `.` → `/app`
- named volume `nestjs_node_modules` → `/app/node_modules`

The host must contain a built `dist/` **or** you rely on the image build output. If the mount hides a missing `dist/`, run `npm run build` on the host or temporarily remove the bind mount for a production-like run.

## Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `APP_PORT` | `8000` | Application port |
| `HOST_PORT` | `8000` | Host port |
| `APP_UID` | `1000` | `nestjs` user UID |
| `APP_USER` | `nestjs` | Username |

## Image notes

The default `node` user on `node:lts-alpine` is removed before creating `APP_USER` to avoid GID conflicts on UID `1000`.

## Commands

```bash
docker compose build
docker compose up
```

## CI

Use `STACK=nestjs` when triggering Jenkins. See [root README](../README.md#cicd-jenkins).
