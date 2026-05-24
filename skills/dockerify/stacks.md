# Stack reference (docker-images)

Paths are relative to this repository root.

## rust

| Item | Value |
|------|--------|
| Template | [rust/](../../rust/) |
| Build | `rust:1.85-alpine` → `alpine:3.21` |
| Runtime CMD | `/app/bin` |
| Jenkins `STACK` | `rust` |

**Required files:** `Cargo.toml`, `src/`, `Cargo.lock` recommended.

**Env:** `APP_BIN` — set when the release binary name is not auto-detected.

**Listen:** binary must bind `0.0.0.0:${APP_PORT}`.

**Dev pitfall:** compose bind-mount replaces `/app/bin` from the image; use `docker run` without mount for prod-like test.

---

## python

| Item | Value |
|------|--------|
| Template | [python/](../../python/) |
| Runtime | Alpine 3.21 + `apk` Python 3, Uvicorn |
| Default CMD | `uvicorn ${APP_MODULE} --host 0.0.0.0 --port ${APP_PORT}` |
| Jenkins `STACK` | `python` |

**Required:** application package (e.g. `app/`).

**Env:** `APP_MODULE` default `app.main:app`.

**Optional:** `requirements.txt` (Uvicorn base already in image).

**Site-packages path:** `/usr/lib/python3.12/site-packages` (Alpine).

---

## nestjs

| Item | Value |
|------|--------|
| Template | [nestjs/](../../nestjs/) |
| Build | `node:lts-alpine`, `npm ci`, `npm run build` |
| Jenkins `STACK` | `nestjs` |

**Required:** `package.json`, `package-lock.json` (for `npm ci`), `src/`, build → `dist/`.

**Env:** `PORT` and `APP_PORT` set in compose.

**main.ts:**

```typescript
await app.listen(Number(process.env.PORT ?? process.env.APP_PORT ?? 8000));
```

**Dev pitfall:** bind mount hides `dist/` unless built on host; volume `nestjs_node_modules` for deps.

---

## static

| Item | Value |
|------|--------|
| Template | [static/](../../static/) |
| Runtime | Nginx 1.27 Alpine, non-root |
| Jenkins `STACK` | `static` |

**Option A:** `index.html` + assets in build context root.

**Option B:** build SPA elsewhere, copy `dist/*` into context before `docker compose build`.

**Config:** [static/nginx/default.conf](../../static/nginx/default.conf) — `__APP_PORT__` at build time.

**Dev:** compose mounts `.` → `/usr/share/nginx/html` (live reload without rebuild).

---

## php

| Item | Value |
|------|--------|
| Template | [php/](../../php/) |
| Runtime | PHP 8.4 FPM + Nginx, Unix socket |
| Jenkins `STACK` | `php` |

**Env / build arg:** `WEB_ROOT` — `public` (Laravel/Symfony) or `.` (plain PHP at root).

**Composer:** runs at image build if `composer.json` present.

**Extensions:** pdo_mysql, zip, intl, opcache, bcmath, pcntl, gd.

**No Apache** — do not add `php:apache` variants unless user overrides.
