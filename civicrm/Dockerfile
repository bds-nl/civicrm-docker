# BASE_IMAGE can be overridden (e.g. by CI) to point at another registry/tag.
# Defaults to the Docker Hub image used by build.sh.
ARG PHP_VERSION=8.4
ARG BASE_IMAGE=ghcr.io/bds-nl/civicrm-base:php${PHP_VERSION}

FROM ${BASE_IMAGE} AS build

ARG CIVICRM_VERSION
ARG CIVICRM_DOWNLOAD_URL="https://download.civicrm.org/civicrm-${CIVICRM_VERSION}-standalone.tar.gz"

RUN curl --fail --location ${CIVICRM_DOWNLOAD_URL} --output civicrm-standalone.tar.gz \
    && tar -xf civicrm-standalone.tar.gz \
    && rm civicrm-standalone.tar.gz

FROM ${BASE_IMAGE}

COPY civicrm.ini /usr/local/etc/php/conf.d/civicrm.ini

RUN curl -LsS https://download.civicrm.org/cv/cv.phar -o /usr/local/bin/cv \
    && chmod +x /usr/local/bin/cv

RUN mkdir /var/www/.cv && chown 33:33 /var/www/.cv

# Copy the downloaded tarball to /var/www/html as the www-data user (uid 33).
# --link makes this an independent, content-addressed layer so the identical
# COPY in nginx/Dockerfile produces the same digest and the registry stores it
# once. Numeric --chown (not the name) is required for --link to be reproducible
# across the two different base images, and must stay in sync between them.
COPY --link --from=build --chown=33:33 /var/www/html/civicrm-standalone /var/www/html

ENV CIVICRM_UF=Standalone
