# PHP 8.5 Docker + Laravel Installation Guide

**Status:** Original procedure documented; proposed corrections are **not applied**.  
**Environment:** Ubuntu host, Docker, PHP 8.5 Apache image, Laravel, Composer.  
**Important:** Review the proposed fixes at the end before running this procedure in production.

## Purpose

Build a custom PHP 8.5 Docker image with the PHP extensions required by Laravel and Composer. This makes the selected extensions and Composer available after container recreation, provided the recreated container uses the custom image.

Do **not** run `composer update` or use `--ignore-platform-reqs` merely to work around missing extensions. An existing `composer.lock` may be valid even if the original container lacks required extensions.

## Step 1 — Create the Dockerfile

On your Ubuntu host:

```bash
mkdir -p ~/docker/php85
nano ~/docker/php85/Dockerfile
```

Original Dockerfile:

```dockerfile
FROM php:8.5-apache

RUN apt-get update && apt-get install -y \
    libicu-dev \
    libpng-dev \
    libjpeg62-turbo-dev \
    libfreetype-dev \
    libzip-dev \
    git \
    unzip \
    && docker-php-ext-configure gd --with-freetype --with-jpeg \
    && docker-php-ext-install -j$(nproc) \
       intl \
       bcmath \
       gd \
       pdo_mysql \
       mysqli \
       zip \
    && a2enmod rewrite \
    && rm -rf /var/lib/apt/lists/*

COPY --from=composer/composer:2-bin /composer /usr/local/bin/composer

WORKDIR /var/www/html
```

Save the file.

## Step 2 — Build the custom image

```bash
cd ~/docker/php85
sudo docker build -t php85-laravel .
```

## Step 3 — Verify PHP extensions

```bash
sudo docker run --rm php85-laravel php -m | grep -Ei 'intl|bcmath|gd|pdo_mysql|mysqli|zip'
```

Expected extension names:

```text
bcmath
gd
intl
mysqli
pdo_mysql
zip
```

## Step 4 — Install Laravel dependencies

Choose the correct application path. Original example:

```bash
cd /var/www/html/hirose85
```

Run Composer inside the custom PHP 8.5 image:

```bash
sudo docker run --rm \
  --user "$(id -u):$(id -g)" \
  -e COMPOSER_HOME=/tmp/composer \
  -e HOME=/tmp \
  -v "$PWD:/app" \
  -w /app \
  php85-laravel \
  composer install --no-interaction --prefer-dist
```

Alternatively, obtain the project through `git clone <repository-url>` before running Composer. The original procedure does not specify a clone destination or repository.

## Step 5 — Recreate the running PHP 8.5 container

> **Caution:** The original commands below forcibly remove the existing `php85` container. Do not run them until its existing volumes, environment, network settings and any data stored outside volumes have been reviewed. The two variants are alternatives, not sequential commands.

Original localhost-only binding:

```bash
sudo docker rm -f php85

sudo docker run -d \
  --name php85 \
  --restart unless-stopped \
  -p 127.0.0.1:8085:80 \
  -v /var/www/html/mann:/var/www/html \
  php85-laravel
```

Original remotely accessible binding:

```bash
sudo docker rm -f php85

sudo docker run -d \
  --name php85 \
  --restart unless-stopped \
  -p 0.0.0.0:8085:80 \
  -v /var/www/html/mann:/var/www/html \
  php85-laravel
```

> **Unresolved:** The application was described as `/var/www/html/hirose85`, but these bind mounts use `/var/www/html/mann:/var/www/html`. This mismatch must be resolved before deployment. The commands are deliberately preserved from the original procedure.

## Step 6 — Verify the container

```bash
sudo docker exec php85 php -v
sudo docker exec php85 composer --version
sudo docker exec php85 php -m | grep -Ei 'intl|bcmath|gd'
```

Check Laravel package requirements from the application directory on the Ubuntu host:

```bash
sudo docker run --rm \
  --user "$(id -u):$(id -g)" \
  -e COMPOSER_HOME=/tmp/composer \
  -e HOME=/tmp \
  -v "$PWD:/app" \
  -w /app \
  php85-laravel \
  composer check-platform-reqs
```

Enter the running container:

```bash
sudo docker exec -it php85 bash
```

Check port bindings:

```bash
sudo docker ps --filter name=php85
```

## Step 7 — Enable Apache `.htaccess` overrides

Original in-container configuration:

```bash
sudo docker exec php85 a2enmod rewrite

sudo docker exec php85 bash -c \
  "printf '<Directory /var/www/html/hirose85/>\nAllowOverride All\nRequire all granted\n</Directory>\n' > /etc/apache2/conf-available/laravel.conf && a2enconf laravel"
```

Validate and restart:

```bash
sudo docker exec php85 apache2ctl configtest
sudo docker restart php85
```

> **Unresolved:** This Apache configuration is lost when the container is recreated. It also does not set Laravel's `public` directory as Apache's `DocumentRoot`. Preserve the original commands for reference, but address both issues in the approved revised Docker image.

## Step 8 — Address deprecated MySQL PDO constant

In PHP 8.5, `PDO::MYSQL_ATTR_SSL_CA` is deprecated. Original code:

```php
PDO::MYSQL_ATTR_SSL_CA => env('MYSQL_ATTR_SSL_CA'),
```

PHP 8.5 alternative:

```php
\Pdo\Mysql::ATTR_SSL_CA => env('MYSQL_ATTR_SSL_CA'),
```

Original suggested compatibility alternative:

```php
(class_exists(\Pdo\Mysql::class)
    ? \Pdo\Mysql::ATTR_SSL_CA
    : \PDO::MYSQL_ATTR_SSL_CA) => env('MYSQL_ATTR_SSL_CA'),
```

The original notes identified `config/database.php` lines 60, 80, 99, 119, 139 and 158; verify line numbers against the actual version of your project before editing.

Clear cached config:

```bash
sudo docker exec -w /var/www/html/hirose85 php85 \
  php artisan config:clear
```

Verify code references:

```bash
sudo docker exec php85 grep -n 'ATTR_SSL_CA' \
  /var/www/html/hirose85/config/database.php
```

> **Unresolved:** `Pdo\Mysql` requires a sufficiently new PHP version, and a mixed-version codebase needs compatibility testing. No PHP application code has been changed by this document.

## Step 9 — Fix Laravel filesystem permissions

These commands aim to address errors writing `storage/logs/laravel.log` and `storage/framework/cache/data` without rebuilding or restarting Docker.

### 9.1 Create storage directories

```bash
sudo docker exec php85 mkdir -p \
  /var/www/html/hirose85/storage/logs \
  /var/www/html/hirose85/storage/framework/cache/data \
  /var/www/html/hirose85/storage/framework/sessions \
  /var/www/html/hirose85/storage/framework/views \
  /var/www/html/hirose85/bootstrap/cache
```

### 9.2 Fix group ownership and permissions

Apache in the official Debian-based PHP Apache image normally uses `www-data` (UID 33).

```bash
sudo docker exec php85 chgrp -R www-data \
  /var/www/html/hirose85/storage \
  /var/www/html/hirose85/bootstrap/cache

sudo docker exec php85 chmod -R ug+rwX \
  /var/www/html/hirose85/storage \
  /var/www/html/hirose85/bootstrap/cache

sudo docker exec php85 find \
  /var/www/html/hirose85/storage \
  /var/www/html/hirose85/bootstrap/cache \
  -type d -exec chmod g+s {} +
```

This avoids `chmod 777`.

### 9.3 Existing log file permissions

Only if the log file already exists:

```bash
sudo docker exec php85 chgrp www-data \
  /var/www/html/hirose85/storage/logs/laravel.log

sudo docker exec php85 chmod 664 \
  /var/www/html/hirose85/storage/logs/laravel.log
```

### 9.4 Clear caches as Apache's user

```bash
sudo docker exec \
  -u www-data \
  -w /var/www/html/hirose85 \
  php85 php artisan optimize:clear
```

If the application uses database or Redis cache, those services must be reachable.

### 9.5 Verify write access

```bash
sudo docker exec -u www-data php85 \
  touch /var/www/html/hirose85/storage/logs/permission-test.log

sudo docker exec -u www-data php85 \
  touch /var/www/html/hirose85/storage/framework/cache/data/permission-test
```

Remove the test files:

```bash
sudo docker exec php85 rm -f \
  /var/www/html/hirose85/storage/logs/permission-test.log \
  /var/www/html/hirose85/storage/framework/cache/data/permission-test
```

### 9.6 Troubleshoot persistent permissions errors

```bash
sudo docker exec php85 ls -ld \
  /var/www/html/hirose85/storage \
  /var/www/html/hirose85/storage/logs \
  /var/www/html/hirose85/storage/framework/cache/data

sudo docker exec php85 ls -l \
  /var/www/html/hirose85/storage/logs/laravel.log
```

Because a Laravel folder is bind-mounted from Ubuntu, permissions changed in the container also affect files on the host.

---

# Review Findings — Proposed Changes Only (Not Applied)

The sections above preserve the originally supplied commands. The following changes are **recommendations** and require approval before being incorporated into a revised deployment procedure.

| Priority | Finding | Proposed correction |
| --- | --- | --- |
| High | Paths disagree (`/var/www/html/hirose85` versus `/var/www/html/mann:/var/www/html`). | Select one consistent host/container layout and adjust all commands together. |
| High | `php:8.5-apache` serves `/var/www/html` by default. | Set Apache `DocumentRoot` to the specific Laravel application's `public` directory and verify its location. |
| High | Apache `a2enconf` changes made with `docker exec` disappear on recreation. | Bake Apache site configuration and rewrite settings into the Docker image. |
| High | `docker rm -f php85` could remove critical container-only settings or data. | Inspect `docker inspect php85`, volumes, networks, variables, and deployment configuration before replacement. |
| High | Laravel 11 support for PHP 8.5 is not guaranteed. | Confirm installed Laravel version and PHP compatibility independently of extension checks. |
| Medium | Not every required PHP extension is established by the original list. | Run `composer check-platform-reqs`; install additional extensions only as proven necessary. |
| Medium | Composer commands depend on the active shell directory and repository contents. | Confirm `pwd`, `composer.json`, and `composer.lock`; ensure project and vendor ownership are suitable. |
| Medium | Bind-mounted permissions may change with host ownership, deployment, or code checkout. | Define a stable ownership/group/ACL approach, optionally with a controlled deployment permissions step. |
| Medium | PHP 8.5 PDO deprecation workaround and cross-version support need testing. | Verify actual minimum PHP version and modify `config/database.php` only as needed. |
| Medium | `0.0.0.0:8085:80` opens a remote listener. | Use `127.0.0.1` behind a reverse proxy unless remote access is explicitly required and protected by firewall rules. |
| Medium | `php:8.5-apache` and Composer 2 tags may shift over time. | Consider version/digest pinning and a deliberate security-patch rebuild procedure. |
| Low | Laravel file permission testing covers logs and cache only. | Also verify Blade views, sessions, `bootstrap/cache`, and application health checks. |

## Sources checked

- [Composer documentation — Docker / binary-only image](https://github.com/composer/composer/blob/main/doc/00-intro.md)
- [Composer documentation — dependency caching and runtime parity](https://getcomposer.org/doc/articles/caching.md)
- [PHP manual — MySQL PDO constants and PHP 8.5 deprecations](https://www.php.net/manual/en/ref.pdo-mysql.php)
- [Laravel framework discussion — Laravel 11 and PHP 8.5](https://github.com/laravel/framework/issues/58467)

**Approval status:** No proposed technical changes applied; this file documents the original procedure and identifies items to correct next.
