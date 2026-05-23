# PHP Laravel

Multi-stage image for **Laravel** with **PHP 8.4 FPM Alpine**, **Nginx**, and communication over a **Unix socket** (`/var/run/php/php-fpm.sock`). No Apache.

## Project requirements

Place a Laravel project in this directory root:

- `composer.json` / `composer.lock`
- `public/` (Nginx document root)
- `artisan`, `app/`, `config/`, etc.

Nginx points to `/var/www/html/public`. Laravel routes use `try_files` + `index.php`.

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

After the first `up` with a Laravel project:

```bash
docker compose exec app php artisan key:generate
docker compose exec app php artisan migrate
```

## CI

Use `STACK=php-laravel` when triggering Jenkins. See [root README](../README.md#cicd).
