# File Permissions Cheat Sheet

## Standard Ownership

| Application | User | Group | Web Root |
|-------------|------|-------|----------|
| WordPress (Nginx) | `www-data` | `www-data` | `/var/www/html` |
| WordPress (Apache) | `www-data` | `www-data` | `/var/www/html` |
| Laravel | `www-data` | `www-data` | `/var/www/html` |
| Next.js (self-host) | `node` / `nextjs` | `nodejs` | `/opt/app` |
| Django/Flask | `www-data` | `www-data` | `/var/www/app` |
| Static sites | `www-data` | `www-data` | `/var/www/html` |

## Permission Templates

### WordPress (Standard)

```bash
# Set ownership
chown -R www-data:www-data /var/www/html

# Directories
find /var/www/html -type d -exec chmod 755 {} \;

# Files
find /var/www/html -type f -exec chmod 644 {} \;

# wp-config.php (restricted)
chmod 600 /var/www/html/wp-config.php

# Uploads (writable)
chmod 775 /var/www/html/wp-content/uploads
```

### WordPress (Hardened)

```bash
# Root owned (read-only for web)
chown -R root:root /var/www/html
chmod -R 755 /var/www/html

# Writeable directories
chown -R www-data:www-data /var/www/html/wp-content/uploads
chmod 775 /var/www/html/wp-content/uploads

# wp-config.php locked down
chmod 600 /var/www/html/wp-content/uploads
```

### Laravel

```bash
# Set ownership
chown -R www-data:www-data /var/www/html

# Standard permissions
find /var/www/html -type d -exec chmod 755 {} \;
find /var/www/html -type f -exec chmod 644 {} \;

# Writeable directories (storage, cache, logs)
chmod -R 775 /var/www/html/storage
chmod -R 775 /var/www/html/bootstrap/cache

# artisan executable
chmod +x /var/www/html/artisan

# .env restricted
chmod 640 /var/www/html/.env
```

### Static Sites

```bash
chown -R www-data:www-data /var/www/html
find /var/www/html -type d -exec chmod 755 {} \;
find /var/www/html -type f -exec chmod 644 {} \;
```

## Permission Reference

| Permission | Numeric | Meaning |
|------------|---------|---------|
| `rwxrwxrwx` | `777` | Everyone can read/write/execute (insecure) |
| `rwxr-xr-x` | `755` | Owner full, others read/execute (directories) |
| `rw-r--r--` | `644` | Owner write, others read (files) |
| `rw-------` | `600` | Owner only (config files, keys) |
| `rwx------` | `700` | Owner only (private directories) |
| `rw-rw-r--` | `664` | Owner+group write, others read |
| `rwxrwxr-x` | `775` | Owner+group full, others read/execute |
| `-----x--x` | `0111` | Execute only (jail directories) |

## Common Fixes

```bash
# Fix all permissions (WordPress)
chown -R www-data:www-data .
find . -type d -exec chmod 755 {} \;
find . -type f -exec chmod 644 {} \;
chmod 600 wp-config.php
```

```bash
# Fix all permissions (Laravel)
chown -R www-data:www-data .
find . -type d -exec chmod 755 {} \;
find . -type f -exec chmod 644 {} \;
chmod -R 775 storage bootstrap/cache
chmod +x artisan
```

```bash
# Fix SSH directory
chmod 700 ~/.ssh
chmod 600 ~/.ssh/id_ed25519
chmod 644 ~/.ssh/id_ed25519.pub
chmod 600 ~/.ssh/authorized_keys
chmod 600 ~/.ssh/config
```

```bash
# Fix Docker socket
chmod 660 /var/run/docker.sock

# Fix logrotate
chmod 644 /etc/logrotate.d/nginx
```

## Permission Issues

| Error | Cause | Fix |
|-------|-------|-----|
| "Permission denied" writing uploads | Uploads dir not writable | `chmod 775 wp-content/uploads` |
| "Failed to open stream" in logs | Storage not writable | `chmod -R 775 storage` |
| "Connection refused" on SSH | `.ssh` wrong permissions | `chmod 700 ~/.ssh` |
| Nginx 403 Forbidden | Directory not readable | `chmod 755 /var/www/html` |
| "is not writable" in admin | wp-config.php read-only | `chmod 600 wp-config.php` |

## Verification

```bash
# Check file types
find /var/www/html -type f -not -perm 644 | head
find /var/www/html -type d -not -perm 755 | head

# Check ownership
find /var/www/html -not -user www-data | head

# Check writable by www-data
sudo -u www-data touch /var/www/html/wp-content/uploads/test.txt && rm /var/www/html/wp-content/uploads/test.txt

# Check SSH permissions
stat -c "%a %n" ~/.ssh/id_ed25519
```
