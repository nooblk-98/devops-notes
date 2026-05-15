# Staging Environment Setup

## Purpose

Create a staging environment that mirrors production for safe testing.

## Clone Production to Staging

### 1. Clone Files

```bash
# From production server, rsync to staging
rsync -avz --delete -e ssh \
  --exclude='wp-config.php' \
  --exclude='.env' \
  --exclude='cache/' \
  --exclude='logs/' \
  /var/www/html/ user@staging-server:/var/www/html/
```

### 2. Clone Database

```bash
# Export production DB
mysqldump -u root -p wordpress > /tmp/wp-prod.sql

# Transfer to staging
rsync -avz /tmp/wp-prod.sql user@staging-server:/tmp/

# Import to staging DB
ssh user@staging-server "mysql -u root -p wordpress_staging < /tmp/wp-prod.sql"
```

### 3. Update URLs

```bash
# On staging server
wp search-replace "example.com" "staging.example.com" --all-tables
wp rewrite flush
```

### 4. Configure wp-config.php

```php
// Staging-specific
define('WP_DEBUG', true);
define('WP_DEBUG_LOG', true);
define('WP_DEBUG_DISPLAY', false);
define('DISALLOW_FILE_EDIT', true);
define('DISABLE_WP_CRON', true);

// Disable emails on staging
define('WP_SITEURL', 'https://staging.example.com');
define('WP_HOME', 'https://staging.example.com');
```

## Docker-Based Staging

```yaml title="docker-compose.staging.yml"
version: "3.9"

services:
  maria-db:
    image: mariadb:10
    environment:
      MYSQL_ROOT_PASSWORD: staging_pass
      MYSQL_DATABASE: wordpress
    volumes:
      - ./db_staging:/var/lib/mysql

  wordpress:
    image: wordpress:php8.2-fpm
    env_file: .env.staging
    volumes:
      - ./html:/var/www/html
    ports:
      - "127.0.0.1:9001:9000"

  nginx:
    image: nginx:alpine
    ports:
      - "8080:80"
    volumes:
      - ./html:/var/www/html
      - ./nginx/staging.conf:/etc/nginx/conf.d/default.conf:ro
```

```bash
docker compose -f docker-compose.staging.yml up -d
```

## Isolate from Production

- **Separate database** — never share DB between staging and production
- **Separate server** — ideally different machine
- **Separate domain** — `staging.example.com` with password protection
- **Disable emails** — prevent staging sending real emails
- **Disable external webhooks** — no payment, no API calls

## Password Protect Staging

### Nginx Basic Auth

```bash
# Create password file
apt install -y apache2-utils
htpasswd -c /etc/nginx/.htpasswd admin
```

```nginx
server {
    listen 80;
    server_name staging.example.com;

    auth_basic "Staging Site";
    auth_basic_user_file /etc/nginx/.htpasswd;

    location / {
        try_files $uri $uri/ /index.php?$args;
    }
}
```

### Cloudflare Access (Better)

1. Dashboard → Access → Applications → Add Application
2. Domain: `staging.example.com`
3. Policy: Email OTP or Google login
4. Only team members can access

## Automate Staging Deploy

```yaml title=".github/workflows/deploy-staging.yml"
name: Deploy to Staging

on:
  push:
    branches: [develop]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Deploy to staging
        run: |
          rsync -avz --delete -e ssh ./ user@staging:/var/www/html/
          ssh user@staging "wp search-replace 'https://example.com' 'https://staging.example.com' --all-tables"
```

## Sync Production Down to Staging

```bash title="/usr/local/bin/sync-prod-to-staging.sh"
#!/bin/bash
PROD_HOST="production-server"
STAGING_HOST="staging-server"
PROD_DB="wordpress"
STAGING_DB="wordpress_staging"
PROD_PATH="/var/www/html"
STAGING_PATH="/var/www/html"

# Sync files
rsync -avz --delete -e ssh \
  --exclude='wp-config.php' \
  --exclude='.env' \
  $PROD_HOST:$PROD_PATH/ $STAGING_PATH/

# Dump and restore DB
ssh $PROD_HOST "mysqldump -u root -p $PROD_DB" | \
  mysql -u root -p $STAGING_DB

# Update URLs
wp search-replace "example.com" "staging.example.com" --all-tables --path=$STAGING_PATH
```

```bash
chmod +x /usr/local/bin/sync-prod-to-staging.sh
0 2 * * 0 /usr/local/bin/sync-prod-to-staging.sh  # Weekly sync
```

## Staging Checklist

- [ ] Database is a copy (not linked to production)
- [ ] URLs updated to staging domain
- [ ] Emails disabled
- [ ] Webhooks/payments disabled
- [ ] Password protected
- [ ] SSL certificate installed
- [ ] Same PHP/DB versions as production
- [ ] Search engines blocked (noindex)
