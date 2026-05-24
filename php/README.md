# PHP

Multi-stage image for **PHP 8.4 FPM Alpine** + **Nginx** over a **Unix socket** (`/var/run/php/php-fpm.sock`). Framework-agnostic (Laravel, Symfony, plain PHP). No Apache.

## What ships here

- `Dockerfile`, `docker-compose.yml`, `.env.example`
- [nginx/default.conf](nginx/default.conf), [php-fpm/zz-docker.conf](php-fpm/zz-docker.conf)
- No application code (add your PHP project in this directory)

## Project requirements

Place your PHP project at this directory root.

| Layout | Document root | `WEB_ROOT` (build arg) |
|--------|---------------|-------------------------|
| Laravel, Symfony | `public/index.php` | `public` (default) |
| Plain PHP at repo root | `index.php` | `.` |

Optional `composer.json` / `composer.lock`: dependencies are installed at **image build** when present.

## Local usage

```bash
cp .env.example .env
docker compose up --build
```

URL: `http://localhost:${HOST_PORT}` (default `8000`).

### Development volumes

Compose mounts `.` → `/var/www/html` so code changes apply without rebuild (reload PHP-FPM / refresh as needed).

## Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `APP_PORT` | `8000` | Nginx port (build arg) |
| `HOST_PORT` | `8000` | Host port |
| `APP_UID` | `1000` | User UID |
| `APP_USER` | `www-data` | PHP-FPM and Nginx user |
| `WEB_ROOT` | `public` | Path under `/var/www/html` (`.` = project root) |

Set `WEB_ROOT` in `.env` and pass it as a compose build arg (see `docker-compose.yml`).

## In-container architecture

- `php-fpm` listens on `/var/run/php/php-fpm.sock`
- Nginx forwards PHP via FastCGI
- Startup: `php-fpm -D`, then `nginx -g 'daemon off;'` (`ENTRYPOINT` cleared)

## PHP extensions

`pdo_mysql`, `zip`, `intl`, `opcache`, `bcmath`, `pcntl`, `gd`

## Commands

```bash
docker compose build
docker compose up
```

## CI

Use `STACK=php` when triggering Jenkins. See [root README](../README.md#cicd-jenkins).
