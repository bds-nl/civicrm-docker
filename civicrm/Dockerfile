# BASE_IMAGE can be overridden (e.g. by CI) to point at another registry/tag.
# Defaults to the Docker Hub image used by build.sh.
ARG BASE_IMAGE=ghcr.io/bds-nl/civicrm-base:latest

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

# Copy the downloaded tarball to /var/www/html as the www-data user
COPY --from=build  --chown=www-data:www-data /var/www/html/civicrm-standalone /var/www/html

ENV CIVICRM_UF=Standalone
