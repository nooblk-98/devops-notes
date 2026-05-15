# Install Certbot & Get SSL Certificate

## Install Certbot

### Ubuntu / Debian

```bash
apt update
apt install -y certbot python3-certbot-nginx
```

### RHEL / Rocky / Alma

```bash
yum install -y epel-release
yum install -y certbot python3-certbot-nginx
```

## Get SSL Certificate (Nginx)

```bash
# Auto-configure Nginx with SSL
certbot --nginx -d example.com -d www.example.com --non-interactive --agree-tos -m admin@example.com
```

### Standalone Mode (no web server running)

```bash
systemctl stop nginx
certbot certonly --standalone -d example.com -d www.example.com --non-interactive --agree-tos -m admin@example.com
systemctl start nginx
```

## Auto-Renewal

```bash
# Test renewal
certbot renew --dry-run

# Certbot adds a systemd timer automatically on most distros
systemctl list-timers | grep certbot
```

Manual renew:

```bash
certbot renew
nginx -t && systemctl reload nginx
```

## Certificate Location

```
/etc/letsencrypt/live/example.com/fullchain.pem
/etc/letsencrypt/live/example.com/privkey.pem
```
