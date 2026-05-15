# Laravel Deployment

**Version:** 1.0  
**Owner:** DevOps Team

## Architecture

```
User → Host Nginx (80/443) → PHP-FPM container (9000) → MariaDB container (3306)
                              → Redis container (6379) [optional]
```

## Prerequisites

- Docker and Docker Compose installed
- Nginx installed on host (`apt install nginx`)
- Node.js 18+ and Composer on deployment host
- Domain DNS pointing to server
- GitHub repository with Actions enabled

## Procedure

### 1. Standard Project Structure

#### Laravel Project Root (Git Repository)

```
laravel-app/
├── app/
├── bootstrap/
├── config/
├── database/
├── public/
├── resources/
├── routes/
├── storage/
├── tests/
├── Dockerfile                  # PHP-FPM image with Composer + Node
├── docker-compose.yml          # App + MariaDB + Redis
├── .env.example                # Environment template
├── .github/
│   └── workflows/
│       └── deploy.yml          # GitHub Actions deploy
├── nginx/
│   └── app.conf                # Nginx config template
└── supervisor/
    └── laravel-worker.conf     # Queue worker config
```

#### Server Deployment Directory

```
/opt/laravel/
├── html/                       # Laravel application files (from git)
│   ├── public/
│   ├── storage/
│   ├── ...
│   ├── Dockerfile
│   └── docker-compose.yml
├── db/                         # MariaDB data volume
├── redis/                      # Redis data volume
├── .env                        # Production environment file
└── nginx/
    └── app.conf                # (optional) Nginx config backup
```

### 2. Create Dockerfile

```dockerfile title="Dockerfile"
FROM php:8.2-fpm AS base

# System dependencies
RUN apt update && apt install -y \
    git curl libpng-dev libjpeg-dev libfreetype6-dev \
    libonig-dev libxml2-dev libzip-dev zip unzip \
    && docker-php-ext-configure gd --with-freetype --with-jpeg \
    && docker-php-ext-install pdo_mysql mbstring exif pcntl bcmath gd zip

# Composer
COPY --from=composer:latest /usr/bin/composer /usr/bin/composer

WORKDIR /var/www/html

# Copy composer files first (layer caching)
COPY composer.json composer.lock ./
RUN composer install --no-dev --optimize-autoloader --no-interaction

# Copy application
COPY . .

# Build frontend
RUN --mount=type=bind,from=node:18,source=/usr/local,target=/usr/local \
    npm ci && npm run build

# Permissions
RUN chown -R www-data:www-data storage bootstrap/cache \
    && chmod -R 775 storage bootstrap/cache

EXPOSE 9000

CMD ["php-fpm"]

# ----- Production stage (multi-stage) -----
FROM base AS production

RUN php artisan optimize \
    && php artisan route:cache \
    && php artisan view:cache \
    && php artisan config:cache

USER www-data
```

### 3. Create Docker Compose File

```yaml title="docker-compose.yml"
services:
  app:
    image: laravel-app:latest
    container_name: laravel-app
    restart: unless-stopped
    build:
      context: .
      dockerfile: Dockerfile
    ports:
      - "127.0.0.1:9000:9000"
    volumes:
      - ./html:/var/www/html
    env_file: .env
    depends_on:
      maria-db:
        condition: service_healthy
    networks:
      - laravel-network

  maria-db:
    image: mariadb:10
    container_name: laravel-db
    restart: unless-stopped
    env_file: .env
    volumes:
      - ./db:/var/lib/mysql
    healthcheck:
      test: ["CMD", "healthcheck.sh", "--connect", "--innodb_initialized"]
      interval: 10s
      timeout: 5s
      retries: 5
    networks:
      - laravel-network

  redis:
    image: redis:alpine
    container_name: laravel-redis
    restart: unless-stopped
    ports:
      - "127.0.0.1:6379:6379"
    volumes:
      - ./redis:/data
    networks:
      - laravel-network

networks:
  laravel-network:
    driver: bridge
```

### 3. Create Dockerfile

```dockerfile title="Dockerfile"
FROM php:8.2-fpm

# System dependencies
RUN apt update && apt install -y \
    git curl libpng-dev libjpeg-dev libfreetype6-dev \
    libonig-dev libxml2-dev libzip-dev zip unzip \
    && docker-php-ext-configure gd --with-freetype --with-jpeg \
    && docker-php-ext-install pdo_mysql mbstring exif pcntl bcmath gd zip

# Composer
COPY --from=composer:latest /usr/bin/composer /usr/bin/composer

# Node.js for frontend build
COPY --from=node:18 /usr/local/bin/node /usr/local/bin/node
COPY --from=node:18 /usr/local/lib/node_modules /usr/local/lib/node_modules
RUN ln -s /usr/local/lib/node_modules/npm/bin/npm-cli.js /usr/local/bin/npm

WORKDIR /var/www/html

COPY . .

RUN composer install --no-dev --optimize-autoloader \
    && npm ci && npm run build \
    && chown -R www-data:www-data storage bootstrap/cache \
    && chmod -R 775 storage bootstrap/cache

EXPOSE 9000

CMD ["php-fpm"]
```

### 4. Create Environment File

```bash title=".env"
APP_NAME=Laravel
APP_ENV=production
APP_KEY=base64:your-generated-key
APP_DEBUG=false
APP_URL=https://example.com

DB_CONNECTION=mysql
DB_HOST=maria-db
DB_PORT=3306
DB_DATABASE=laravel
DB_USERNAME=laravel_user
DB_PASSWORD=strong_password

REDIS_HOST=redis
REDIS_PORT=6379
REDIS_PASSWORD=null

SESSION_DRIVER=redis
CACHE_STORE=redis
QUEUE_CONNECTION=redis
```

!!! warning "Generate APP_KEY"
    Run `php artisan key:generate --show` locally and paste the output into `.env`.

### 5. Configure Nginx on Host

```nginx title="/etc/nginx/sites-available/example.com"
server {
    listen 80;
    server_name example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name example.com;

    root /opt/laravel/html/public;
    index index.php;

    ssl_certificate /etc/letsencrypt/live/example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location ~ \.php$ {
        fastcgi_pass 127.0.0.1:9000;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
        fastcgi_param PATH_INFO $fastcgi_path_info;
    }

    location ~ /\.ht {
        deny all;
    }
}
```

Enable site:

```bash
ln -s /etc/nginx/sites-available/example.com /etc/nginx/sites-enabled/
nginx -t && systemctl reload nginx
```

### 6. SSL Certificate

```bash
certbot --nginx -d example.com --non-interactive --agree-tos -m admin@example.com
```

### 7. Build & Start

```bash
# Build image
docker compose build

# Start containers
docker compose up -d

# Check status
docker compose ps
```

Expected output:

```
NAME          IMAGE               STATUS   PORTS
laravel-app   laravel-app:latest  Up       127.0.0.1:9000->9000/tcp
laravel-db    mariadb:10          Up       3306/tcp
laravel-redis redis:alpine        Up       127.0.0.1:6379->6379/tcp
```

### 8. Run Migrations

```bash
docker compose exec app php artisan migrate --force
docker compose exec app php artisan storage:link
```

### 9. Post-Deployment Verification

```bash
# Health check
curl -I https://example.com

# Laravel-specific
docker compose exec app php artisan about

# Check logs
docker compose logs app --tail=50
tail -f /var/log/nginx/error.log
```

## GitHub Actions Deployment

### Repository Secrets

Add these in GitHub → Settings → Secrets:

| Secret | Value |
|--------|-------|
| `DEPLOY_HOST` | Server IP |
| `DEPLOY_USER` | SSH user |
| `SSH_PRIVATE_KEY` | Private SSH key |
| `APP_KEY` | Laravel APP_KEY |

### Workflow File

```yaml title=".github/workflows/deploy.yml"
name: Deploy Laravel

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Deploy via SSH
        uses: appleboy/ssh-action@v1
        with:
          host: ${{ secrets.DEPLOY_HOST }}
          username: ${{ secrets.DEPLOY_USER }}
          key: ${{ secrets.SSH_PRIVATE_KEY }}
          script: |
            cd /opt/laravel

            # Pull latest code
            git pull origin main

            # Copy .env (managed separately)
            cp .env.production .env

            # Build new image
            docker compose build

            # Run migrations (before switch)
            docker compose run --rm app php artisan migrate --force

            # Recreate containers
            docker compose up -d --force-recreate

            # Cleanup old images
            docker image prune -f

            # Verify
            curl -f https://example.com/health
```

### Zero-Downtime Deploy (Alternative)

```yaml title=".github/workflows/deploy-zd.yml"
name: Zero-Downtime Deploy

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Build image
        run: docker build -t laravel-app:${{ github.sha }} .

      - name: Push to registry
        run: |
          docker tag laravel-app:${{ github.sha }} ${{ secrets.REGISTRY }}/laravel-app:latest
          docker push ${{ secrets.REGISTRY }}/laravel-app:latest

      - name: Deploy
        uses: appleboy/ssh-action@v1
        with:
          host: ${{ secrets.DEPLOY_HOST }}
          username: ${{ secrets.DEPLOY_USER }}
          key: ${{ secrets.SSH_PRIVATE_KEY }}
          script: |
            cd /opt/laravel

            # Pull new image
            docker compose pull app

            # Migrate DB
            docker compose run --rm app php artisan migrate --force

            # Recreate app only (no downtime for other services)
            docker compose up -d --no-deps app
```

## Maintenance

### Artisan Commands

```bash
# Cache
docker compose exec app php artisan config:cache
docker compose exec app php artisan route:cache
docker compose exec app php artisan view:cache

# Clear cache (after updates)
docker compose exec app php artisan optimize:clear

# Queue worker
docker compose exec app php artisan queue:work --daemon

# Schedule (add to cron)
echo "* * * * * docker compose exec app php artisan schedule:run >> /dev/null 2>&1" | crontab -
```

### Backup

```bash
# Database
docker compose exec maria-db mysqldump -u laravel_user -p laravel > backup-$(date +%Y%m%d).sql

# Files (uploads, logs)
tar czf storage-$(date +%Y%m%d).tar.gz html/storage
```

### Update

```bash
git pull origin main
docker compose build app
docker compose run --rm app php artisan migrate --force
docker compose up -d --force-recreate app
```

## Troubleshooting

| Symptom | Cause | Fix |
|---------|-------|-----|
| 502 Bad Gateway | PHP-FPM not running | `docker compose restart app` |
| 403 Forbidden | Wrong root path | Ensure `root` points to `/opt/laravel/html/public` |
| "No application encryption key" | APP_KEY missing | Run `php artisan key:generate` and update `.env` |
| Class not found | Composer autoload stale | `docker compose exec app composer dump-autoload` |
| SQLSTATE[HY000] [2002] | DB container not ready | Wait for healthcheck, or increase `depends_on` retries |
| npm/node not found | Build deps missing | Check Dockerfile installs Node |

## Verification

- [ ] App image built and pushed
- [ ] Containers running (`docker compose ps`)
- [ ] Nginx proxies to FPM on `127.0.0.1:9000`
- [ ] SSL certificate valid
- [ ] Migrations run
- [ ] Cache optimized (config, route, view)
- [ ] Queue worker running
- [ ] Scheduler cron set up
- [ ] GitHub Actions deploy succeeds
- [ ] Health endpoint responds 200
