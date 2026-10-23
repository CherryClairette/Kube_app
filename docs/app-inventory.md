# Docker-compose app inventory (ticket 01)

Source: `docker-compose.yaml`, `Dockerfile` (checked 2026-10-23).

## Services

| Service | Image / source | Ports | Role |
|---|---|---|---|
| `traefik` | `traefik:v3.0` (Docker Hub) | 80, 8082 (metrics) | Reverse proxy, Prometheus metrics on :8082 |
| `app` | built from `Dockerfile` (base `php:8.2.8-apache`, plus `composer:latest` copied in) | 8081 -> 80 | Laravel app, Apache docroot `public/` |
| `db` | `mysql:8.0` (Docker Hub) | 3306 (internal) | MySQL, db `app_database`, user `app_user` |

## Database

- MySQL 8.0, volume `db_data`, healthcheck via `mysqladmin ping`.
- Schema: Laravel migrations (`php artisan migrate`). Seed: `php artisan db:seed` (10 `Counter` rows).
- Credentials are hardcoded in compose (`app_user` / `app_password`) and must move to Kubernetes Secrets.
- Compose mounts `./mysql-init`, which does not exist in the repo (optional, harmless).

## App behaviour

- `GET /api/counter/add` inserts a row with `count = 1`, returns `{"value": sum}`.
- `GET /api/counter/count` returns `{"value": sum}`.
- Env needed: `APP_KEY`, `APP_ENV`, `APP_DEBUG`, `DB_*`.

## arm64 check (manifests inspected with `docker manifest inspect`)

| Image | arm64 |
|---|---|
| `traefik:v3.0` | yes |
| `php:8.2.8-apache` | yes |
| `composer:latest` | yes |
| `mysql:8.0` | yes |

No gap in the compose stack. Still to check: every image of each Helm chart adopted later (Argo CD, Harbor, etc.).

## Flags

- The app image must be built for arm64 (`docker buildx build --platform linux/arm64`) before pushing to Harbor.
- `composer:latest` is unpinned; pin it for reproducible builds.
- `Dockerfile` comment says PHP 7.4, the image is 8.2.8.
- README mentions a `sample-app` folder that does not exist.
