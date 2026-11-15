FROM serversideup/php:8.5.12-cli-v5.1.1 AS base

USER root

RUN install-php-extensions exif gd intl

FROM base AS composer-dependencies

RUN docker-php-serversideup-dep-install-debian git

COPY composer.json composer.lock ./

RUN composer install --no-autoloader --no-dev --no-interaction --no-scripts --prefer-dist

FROM base AS final

ENV AUTORUN_ENABLED=true \
    AUTORUN_LARAVEL_MIGRATION=false \
    AUTORUN_LARAVEL_STORAGE_LINK=false \
    PHP_OPCACHE_ENABLE=1

COPY --chown=www-data:www-data . .
COPY --chown=www-data:www-data --from=composer-dependencies /var/www/html/vendor ./vendor

USER www-data

RUN composer install --no-dev --no-interaction --optimize-autoloader --prefer-dist \
    && composer check-platform-reqs --no-dev

CMD ["php", "/var/www/html/artisan", "schedule:work"]
