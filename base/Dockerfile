# PHP_VERSION can be overridden (e.g. by CI). Trixie base is kept fixed.
ARG PHP_VERSION=8.4
FROM php:${PHP_VERSION}-fpm-trixie

# Install required packages (including those required for PHP extensions).
#
# Please document the reason for each installed package below.
#
# libfreetype-dev, libjpeg62-turbo-dev, libpng-dev, libwebp-dev are required for the PHP gd extension
# libicu is required for the PHP intl extension
# libzip-dev is required for the PHP zip extension
# imagemagick, libmagickwand-dev are required for PHP imagick extension
# mariadb-client is required for cv cli

RUN set -eux; \
    apt-get update && \
    apt-get install -y --no-install-recommends \
    libfreetype-dev \
    libicu-dev \
    libjpeg62-turbo-dev \
    libpng-dev \
    libwebp-dev \
    libzip-dev \
    imagemagick \
    libmagickwand-dev \
    mariadb-client \
    && \
    rm -rf /var/lib/apt/lists/*

# Configure and install php extensions.
#
# Based on https://docs.civicrm.org/installation/en/latest/general/requirements/#php-extensions
#
# Tip: you can check what extensions have been installed with:
# `php -r 'echo implode("\n",get_loaded_extensions());'`.
RUN set -eux && \
    pecl install imagick && \
    docker-php-ext-enable imagick

RUN set -eux && \
    yes '' | pecl install redis && \
    docker-php-ext-enable redis

RUN set -eux && \
    docker-php-ext-configure gd --with-freetype --with-jpeg --with-webp && \
    docker-php-ext-install -j$(nproc) gd

RUN set -eux && \
    docker-php-ext-install bcmath && \
    docker-php-ext-install intl && \
    docker-php-ext-install mysqli && \
    docker-php-ext-install pdo_mysql && \
    docker-php-ext-install zip
