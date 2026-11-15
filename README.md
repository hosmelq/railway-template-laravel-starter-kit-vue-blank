# Laravel with Vue (without authentication)

This application is generated with the official Laravel installer. It includes source-built Docker images for Railway, with MySQL, PostgreSQL or MariaDB and optional Redis. It uses Laravel's default development tooling: SQLite, Pest, Boost and npm.

## Deploy on Railway

- [MySQL](https://railway.com/deploy/laravel-vue-blank-mysql)
- [MySQL with Redis](https://railway.com/deploy/laravel-vue-blank-mysql-redis)
- [PostgreSQL](https://railway.com/deploy/laravel-vue-blank-postgresql)
- [PostgreSQL with Redis](https://railway.com/deploy/laravel-vue-blank-postgresql-redis)
- [MariaDB](https://railway.com/deploy/laravel-vue-blank-mariadb)
- [MariaDB with Redis](https://railway.com/deploy/laravel-vue-blank-mariadb-redis)

## Services

- **APP** serves HTTP on port 8080 with FrankenPHP in classic mode, runs migrations at startup, and exposes `/up` for health checks.
- **Queue** runs `php artisan queue:work`.
- **Scheduler** runs `php artisan schedule:work`.
- **Database** stores application data. Without Redis, Laravel also uses it for cache, sessions and queued jobs.
- **Redis**, when included, stores cache, sessions and queued jobs.
- **Bucket** stores uploaded files through Laravel's `s3` disk.

Queue and Scheduler never run migrations. APP must finish its migrations before workers can use a new database.

Railway builds each service from this repository's `main` branch. Set `RAILWAY_DOCKERFILE_PATH` to `/docker/app/Dockerfile`, `/docker/queue/Dockerfile`, or `/docker/scheduler/Dockerfile`. There are no prebuilt application images, Octane workers, or Inertia SSR service.

Set `APP_ENV=production`, `APP_DEBUG=false`, a shared `APP_KEY`, and the public `APP_URL` on all three services. Use `DB_CONNECTION=mysql`, `pgsql`, or `mariadb` with `DB_URL`. Set `LOG_CHANNEL=stderr` and `LOG_STDERR_FORMATTER=\Monolog\Formatter\JsonFormatter` for structured application logs.

Set `APP_URL` to the complete public HTTPS URL on Railway. The generated `bootstrap/app.php` includes `$middleware->trustProxies(at: '*');` so Laravel trusts Railway's `X-Forwarded-Proto` and `X-Forwarded-For` headers for HTTPS links, redirects and client IP addresses. An HTTP `APP_URL` also works for local Docker use.

For Redis, set `CACHE_STORE=redis`, `QUEUE_CONNECTION=redis`, `SESSION_DRIVER=redis`, and `REDIS_URL` on all three services. Otherwise leave the installer's database-backed defaults.

For files, set `FILESYSTEM_DISK=s3` and the bucket's `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`, `AWS_DEFAULT_REGION`, `AWS_BUCKET`, and `AWS_ENDPOINT`. Current Railway buckets use virtual-hosted URLs. Older buckets can require `AWS_USE_PATH_STYLE_ENDPOINT=true`; use the URL style shown in the bucket's Credentials tab. The bucket stays private; use temporary URLs when downloads need browser access.

## Local development

Use PHP 8.5, Composer, Node 22.18+, 24.11+ or 26+, npm, and SQLite.

```sh
git clone https://github.com/hosmelq/railway-template-laravel-starter-kit-vue-blank.git
cd railway-template-laravel-starter-kit-vue-blank
```

Then run:

```sh
composer setup
composer run dev
```

The local `.env` uses SQLite and local files. Keep `.env`, database files and credentials out of Git. Run the application's tests with `composer test`.

## Mail

Mail uses `log` by default. The application sends no mail out of the box. On Railway, mail sent during a request appears in the **APP logs** and queued mail in the **Queue logs**. Locally, look in `storage/logs/laravel.log`.

Railway blocks outbound SMTP below the Pro plan. For real delivery, configure an HTTPS API mail provider such as Resend, Postmark, or Mailgun: install its Laravel transport dependency, set `MAIL_MAILER`, add the provider's API credentials, and configure `MAIL_FROM_ADDRESS` and `MAIL_FROM_NAME`. Keep secrets in Railway variables. See [Laravel's mail documentation](https://laravel.com/docs/mail).

## Generated source

This repository is generated from [hosmelq/railway-template-laravel](https://github.com/hosmelq/railway-template-laravel). The generated tree includes the S3 adapter, Dockerfiles, `.dockerignore`, README and the one-line proxy configuration in `bootstrap/app.php`. Its monthly workflow replaces the generated tree only after all six database/Redis combinations pass. The existing Git history is preserved. Fork or clone the repository for your own application; changes made directly to this generated repository can be replaced on the next regeneration.
