# PHP

Multi-stage image for **PHP 8.4 FPM Alpine** with **Nginx** and communication over a **Unix socket** (`/var/run/php/php-fpm.sock`). Framework-agnostic (Laravel, Symfony, plain PHP, etc.). No Apache.

## Project requirements

Place your PHP project in this directory root.

Typical layouts:

| Layout | Document root | `WEB_ROOT` |
|--------|---------------|------------|
| Laravel, Symfony | `public/index.php` | `public` (default) |
| Plain PHP at repo root | `index.php` | `.` |

Optional: `composer.json` / `composer.lock` for Composer dependencies (installed at build time when present).

## Local usage

```bash
cp .env.example .env
docker compose up --build
```

Access at `http://localhost:${HOST_PORT}` (default `8000`).

Compose mounts the project at `/var/www/html` for development.

## Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `APP_PORT` | `8000` | Nginx port (build arg) |
| `HOST_PORT` | `8000` | Host port |
| `APP_UID` | `1000` | User UID |
| `APP_USER` | `www-data` | PHP-FPM and Nginx user |
| `WEB_ROOT` | `public` | Web root path under `/var/www/html` (use `.` for project root) |

## In-container architecture

- `php-fpm` listens on `/var/run/php/php-fpm.sock`
- `nginx` proxies FastCGI to the socket
- The container starts PHP-FPM in the background, then Nginx in the foreground (`CMD`)

## Installed PHP extensions

`pdo_mysql`, `zip`, `intl`, `opcache`, `bcmath`, `pcntl`, `gd`

## Manual build

```bash
docker compose build
docker compose up
```

## CI

Use `STACK=php` when triggering Jenkins. See [root README](../README.md#cicd).
