# civicrm-docker

Docker images for running [CiviCRM Standalone](https://docs.civicrm.org/installation/en/latest/standalone/), built multi-arch (`linux/amd64`, `linux/arm64`) and published to the GitHub Container Registry, plus a Docker Compose stack for running them locally.

This repo contains only the Dockerfiles, configuration, and CI — CiviCRM itself is downloaded from civicrm.org at image-build time.

## Images

Three images form a chain, each building on the previous one:

| Image | Source dir | Base | Contents |
|-------|-----------|------|----------|
| `ghcr.io/bds-nl/civicrm-base` | `base/` | `php:<ver>-fpm-trixie` | PHP-FPM + the extensions CiviCRM requires (intl, gd, zip, bcmath, mysqli, pdo_mysql, imagick, redis) and their system libs |
| `ghcr.io/bds-nl/civicrm-fpm` | `civicrm/` | `civicrm-base:latest` | CiviCRM Standalone unpacked into `/var/www/html`, the `cv` CLI, and PHP limits (`civicrm.ini`). The PHP-FPM app container. |
| `ghcr.io/bds-nl/civicrm-nginx` | `nginx/` | `nginx:alpine` | nginx config + the same `/var/www/html` web root, copied from the fpm image, to serve static assets |

The `fpm` and `nginx` images share the CiviCRM web root as a single deduplicated layer (`COPY --link` with a numeric `--chown`, identical in both Dockerfiles), so the registry stores those bytes once.

## Running locally

```bash
docker compose up
```

CiviCRM is then served at **http://localhost:8080**.

The stack (`docker-compose.yaml`) runs three services:

- **civicrm** — the PHP-FPM container (runs as `33:33`). Owns the published port.
- **nginx** — shares the civicrm container's network namespace (`network_mode: service:civicrm`), so it reaches FPM at `127.0.0.1:9000`.
- **db** — MariaDB 11.

## Building & publishing

Builds run in GitHub Actions and push to GHCR. There are three workflows:

- **`poll.yml`** — daily cron (and manual). Reads the current stable version from [`latest.civicrm.org/stable.php`](https://latest.civicrm.org/stable.php); if no `civicrm-fpm` image exists for that version yet, it builds it. **This is the normal way new releases get built.**
- **`build-images.yml`** — builds and pushes `fpm` then `nginx` (nginx builds `FROM` fpm) for a given `civicrm_version`. Invoked by the poller via `workflow_call`, or run manually from the Actions tab. `fpm` builds `FROM civicrm-base:latest`.
- **`build-base.yml`** — **manual only**, takes a `php_version` input. The base image rarely changes, so it is not rebuilt per CiviCRM release. Re-running it updates what subsequent `fpm` builds pull (via `civicrm-base:latest`). Run this first when bumping PHP, then build a release.

All images are tagged with their version and `latest`.

## nginx routing

`nginx/default.conf` is intentionally locked down:

- **Single PHP entry point:** only `/index.php` is passed to FPM; everything else is rewritten to it with no `try_files`, so source files (`civicrm.settings.php`, `composer.json`, `/.git`, `/vendor`) are never served as raw static content.
- A strict allowlist of static extensions is served directly (`.html` is included for the Mosaico email editor).
- `/public/` serves user uploads. The real client IP is recovered from `X-Forwarded-For` (trusting private ranges) and HTTPS state is forwarded to FPM from an upstream proxy.
