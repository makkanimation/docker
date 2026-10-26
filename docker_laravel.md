## Address deprecated MySQL PDO constant

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

# Fix Git safe directory
## Run this inside your PHP 8.5 Docker container:
```bash
git config --global --add safe.directory /var/www/html/hirose85
git status
git branch -a
git remote -v
```
Clear cached config:

```bash
sudo docker exec -w /var/www/html/hirose85 php85 \
  php artisan config:clear

  
> **Unresolved:** `Pdo\Mysql` requires a sufficiently new PHP version, and a mixed-version codebase needs compatibility testing. No PHP application code has been changed by this document.

## Step 9 — Fix Laravel filesystem permissions

These commands aim to address errors writing `storage/logs/laravel.log` and `storage/framework/cache/data` without rebuilding or restarting Docker.

### Create storage directories

```bash
sudo docker exec php85 mkdir -p \
  /var/www/html/hirose85/storage/logs \
  /var/www/html/hirose85/storage/framework/cache/data \
  /var/www/html/hirose85/storage/framework/sessions \
  /var/www/html/hirose85/storage/framework/views \
  /var/www/html/hirose85/bootstrap/cache
```

### Fix group ownership and permissions

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

### Existing log file permissions

Only if the log file already exists:

```bash
sudo docker exec php85 chgrp www-data \
  /var/www/html/hirose85/storage/logs/laravel.log

sudo docker exec php85 chmod 664 \
  /var/www/html/hirose85/storage/logs/laravel.log
```

### Clear caches as Apache's user

```bash
sudo docker exec \
  -u www-data \
  -w /var/www/html/hirose85 \
  php85 php artisan optimize:clear
```

If the application uses database or Redis cache, those services must be reachable.

### Verify write access

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

### Troubleshoot persistent permissions errors

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
