ARG PHP_VERSION=8.4
ARG CIVICRM_VERSION

# FPM_IMAGE can be overridden (e.g. by CI) to point at another registry.
# Defaults to the Docker Hub image used by build.sh.
ARG FPM_IMAGE=ghcr.io/bds-nl/civicrm-fpm:${CIVICRM_VERSION}-php${PHP_VERSION}

FROM ${FPM_IMAGE} AS build

FROM nginx:alpine

COPY nginx.conf /etc/nginx/nginx.conf
COPY default.conf /etc/nginx/conf.d/default.conf

# Must mirror the COPY in civicrm/Dockerfile exactly (--link, numeric --chown,
# same dest path) so both images share one deduplicated layer in the registry.
COPY --link --from=build --chown=33:33 /var/www/html /var/www/html
