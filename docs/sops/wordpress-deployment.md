---
tags:
  - wordpress
  - docker
  - nginx
  - mariadb
  - deployment
---

# WordPress Deployment SOP (Docker)

**Version:** 1.0  
**Last Updated:** 2024-01-01  
**Owner:** DevOps Team

## Purpose

Standardize the deployment of WordPress sites using Docker containers with WordPress PHP-FPM and MariaDB 10. Nginx is installed directly on the host and proxies to the FPM container.

## Architecture

```
User → Host Nginx (ports 80/443) → WordPress PHP-FPM container (port 9000) → MariaDB 10 container (port 3306)
```

## Prerequisites

- Docker and Docker Compose installed
- Nginx installed on host (`apt install nginx` or `yum install nginx`)
- Domain name with DNS pointing to server
- Server with minimum 2GB RAM, 20GB disk
- Ports 80 and 443 open on firewall

## Procedure

### 1. Install Nginx on Host

```bash
# Debian / Ubuntu
apt update && apt install -y nginx certbot python3-certbot-nginx  # (1)

# RHEL / Rocky / Alma
yum install -y nginx certbot python3-certbot-nginx
```

1.  Nginx is installed on the host (not in Docker) and proxies to the PHP-FPM container on port 9000.

Enable and start:

```bash
systemctl enable --now nginx
```

### 2. Directory Structure

```bash
mkdir -p /opt/wordpress/{html,db}
cd /opt/wordpress
```

### 3. Create Docker Compose File

```yaml title="docker-compose.yml"
version: "3.9"

services:
  maria-db:
    image: mariadb:10
    container_name: wp-db
    restart: unless-stopped
    env_file: .env
    volumes:
      - ./db:/var/lib/mysql
    ports:
      - "127.0.0.1:3307:3306"
    networks:
      - wp-network

  wordpress:
    image: wordpress:php8.2-fpm
    container_name: wp-app
    restart: unless-stopped
    env_file: .env
    ports:
      - "127.0.0.1:9000:9000"
    volumes:
      - ./html:/var/www/html
    depends_on:
      - maria-db
    networks:
      - wp-network

networks:
  wp-network:
    driver: bridge
```

!!! note "Port Binding"
    WordPress FPM container exposes port `9000` to the host at `127.0.0.1:9000`. Host Nginx connects to this port via `fastcgi_pass 127.0.0.1:9000`.

### 4. Create Environment File

```bash title=".env"
# Database
MYSQL_DATABASE=wordpress
MYSQL_USER=wpuser
MYSQL_PASSWORD=strong_password_here
MYSQL_ROOT_PASSWORD=strong_root_password_here

# WordPress
WORDPRESS_DB_HOST=maria-db:3306
WORDPRESS_DB_USER=wpuser
WORDPRESS_DB_PASSWORD=strong_password_here
WORDPRESS_DB_NAME=wordpress
```

!!! warning "Secrets"
    Never commit `.env` to version control. Use a secrets manager in production.

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

    root /opt/wordpress/html;
    index index.php;

    ssl_certificate /etc/letsencrypt/live/example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem;

    location / {
        try_files $uri $uri/ /index.php?$args;
    }

    location ~ \.php$ {
        fastcgi_pass 127.0.0.1:9000;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
    }

    location ~ /\.ht {
        deny all;
    }
}
```

Enable the site:

```bash
ln -s /etc/nginx/sites-available/example.com /etc/nginx/sites-enabled/
nginx -t && systemctl reload nginx
```

### 6. Start Containers

```bash
docker compose up -d
docker compose ps
```

Expected output — two containers running:

```
NAME      IMAGE                   STATUS   PORTS
wp-app    wordpress:php8.2-fpm    Up       127.0.0.1:9000->9000/tcp
wp-db     mariadb:10              Up       127.0.0.1:3307->3306/tcp
```

### 7. Configure SSL (Let's Encrypt)

```bash
certbot --nginx -d example.com --non-interactive --agree-tos -m admin@example.com
```

This automatically updates the Nginx config with SSL directives.

### 8. Set File Permissions

```bash
# WordPress container runs as www-data (uid 33)
chown -R 33:33 /opt/wordpress/html
```

### 9. Post-Deployment Verification

```bash
# Check containers are healthy
docker compose ps

# Verify Nginx config
nginx -t

# Verify WordPress responds
curl -I https://example.com

# Check PHP-FPM status (via container)
docker compose exec wordpress php -v

# Check database connection
docker compose exec wordpress php -r "new PDO('mysql:host=maria-db;dbname=wordpress', 'wpuser', 'strong_password_here'); echo 'DB OK\n';"

# View container logs
docker compose logs --tail=50 -f

# Check host Nginx logs
tail -f /var/log/nginx/access.log /var/log/nginx/error.log
```

## Common Maintenance Tasks

### Backup Database

```bash
docker compose exec maria-db mysqldump -u wpuser -p wordpress > backup-$(date +%Y%m%d).sql
```

### Backup Files

```bash
tar czf wp-files-$(date +%Y%m%d).tar.gz html/
```

### Update WordPress

```bash
# Pull latest images
docker compose pull wordpress

# Recreate containers
docker compose up -d --force-recreate
```

## Troubleshooting

| Symptom | Cause | Fix |
|---------|-------|-----|
| 502 Bad Gateway | Nginx cannot reach FPM container | Verify `docker compose ps`, check `127.0.0.1:9000` is listening |
| 502 Bad Gateway | File permissions mismatch | `chown -R 33:33 /opt/wordpress/html` |
| Database connection error | Wrong credentials in `.env` | Verify `.env` values, restart stack `docker compose down && docker compose up -d` |
| Permission denied writing uploads | www-data ownership missing | `chown -R 33:33 /opt/wordpress/html/wp-content/uploads` |

## Verification

- [ ] Nginx installed and running on host (`systemctl status nginx`)
- [ ] Both containers running (`docker compose ps`)
- [ ] Nginx can reach FPM on `127.0.0.1:9000`
- [ ] Site loads over HTTPS with valid SSL
- [ ] WordPress admin dashboard accessible at `/wp-admin`
- [ ] Database connection working
- [ ] Backups configured and tested
- [ ] Logs show no errors
