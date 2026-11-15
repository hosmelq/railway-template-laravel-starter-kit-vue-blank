FROM serversideup/php:8.5.12-frankenphp-v5.1.1 AS base

USER root

RUN install-php-extensions exif gd intl

FROM base AS composer-dependencies

RUN docker-php-serversideup-dep-install-debian git

COPY composer.json composer.lock ./

RUN composer install --no-autoloader --no-dev --no-interaction --no-scripts --prefer-dist

FROM node:24.22.3-bookworm-slim AS node-runtime

FROM base AS npm-dependencies

COPY --from=node-runtime /usr/local/bin/node /usr/local/bin/node
COPY --from=node-runtime /usr/local/lib/node_modules /usr/local/lib/node_modules

RUN ln -s /usr/local/lib/node_modules/npm/bin/npm-cli.js /usr/local/bin/npm \
    && ln -s /usr/local/lib/node_modules/npm/bin/npx-cli.js /usr/local/bin/npx

COPY .npmrc package.json package-lock.json ./

RUN npm ci

FROM npm-dependencies AS frontend-build

# Railway exposes service variables as build args. Only public asset settings
# are needed here; runtime credentials must not be baked into the image.
ARG APP_NAME=Laravel
ARG VITE_APP_NAME

COPY . .
COPY --from=composer-dependencies /var/www/html/vendor ./vendor

# Wayfinder runs Artisan during the Vite build.
RUN composer install --no-dev --no-interaction --optimize-autoloader --prefer-dist \
    && VITE_APP_NAME="${VITE_APP_NAME:-${APP_NAME}}" npm run build

FROM base AS final

ENV AUTORUN_ENABLED=true \
    AUTORUN_LARAVEL_MIGRATION=true \
    AUTORUN_LARAVEL_STORAGE_LINK=false \
    HEALTHCHECK_PATH=/up \
    PHP_OPCACHE_ENABLE=1

COPY --chown=www-data:www-data . .
COPY --chown=www-data:www-data --from=composer-dependencies /var/www/html/vendor ./vendor
COPY --chown=www-data:www-data --from=frontend-build /var/www/html/public/build ./public/build

USER www-data

RUN composer install --no-dev --no-interaction --optimize-autoloader --prefer-dist \
    && composer check-platform-reqs --no-dev

# The base image runs classic FrankenPHP on port 8080 and checks HEALTHCHECK_PATH.
