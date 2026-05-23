# NestJS

Multi-stage image for **NestJS** APIs on `node:lts-alpine`: `npm ci`, `npm run build`, and a runtime with production dependencies only.

## Project requirements

Place a standard NestJS project in this directory root:

- `package.json` / `package-lock.json`
- `src/`
- A `build` script that produces `dist/`

In `src/main.ts`, listen on the port from the environment:

```typescript
await app.listen(process.env.PORT ?? process.env.APP_PORT ?? 8000);
```

Compose sets `PORT` and `APP_PORT` from `.env`.

## Local usage

```bash
cp .env.example .env
docker compose up --build
```

Access at `http://localhost:${HOST_PORT}` (default `8000`).

The `nestjs_node_modules` volume preserves `node_modules` when mounting local source code.

## Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `APP_PORT` | `8000` | Application port |
| `HOST_PORT` | `8000` | Host port |
| `APP_UID` | `1000` | `nestjs` user UID |
| `APP_USER` | `nestjs` | Username |

## Manual build

```bash
docker compose build
docker compose up
```

## CI

Use `STACK=nestjs` when triggering Jenkins. See [root README](../README.md#cicd).
