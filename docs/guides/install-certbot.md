# Certbot & SSL Certificate Guide

Complete guide to installing and using Certbot for SSL/TLS certificates with Let's Encrypt. Covers Nginx, auto-renewal, Docker, wildcard certificates, and all common commands.

## Installation

### Ubuntu / Debian

```bash
apt update
apt install -y certbot python3-certbot-nginx python3-certbot-dns-cloudflare
```

### RHEL / Rocky / Alma

```bash
dnf install -y epel-release
dnf install -y certbot python3-certbot-nginx python3-certbot-dns-cloudflare
```

### Arch Linux

```bash
pacman -S certbot certbot-nginx
```

### Docker

```bash
alias certbot='docker run -it --rm \
  -v /etc/letsencrypt:/etc/letsencrypt \
  -v /var/lib/letsencrypt:/var/lib/letsencrypt \
  -v /var/log/letsencrypt:/var/log/letsencrypt \
  certbot/certbot'
```

For Nginx integration with Docker, mount webroot:

```bash
alias certbot='docker run -it --rm \
  -v /etc/letsencrypt:/etc/letsencrypt \
  -v /var/lib/letsencrypt:/var/lib/letsencrypt \
  -v /var/log/letsencrypt:/var/log/letsencrypt \
  -v /var/www/html:/var/www/html \
  certbot/certbot'
```

## Basic Usage

### Obtain Certificate with Nginx Plugin (Auto-Configure)

```bash
certbot --nginx -d example.com -d www.example.com \
  --non-interactive --agree-tos -m admin@example.com
```

This automatically modifies Nginx config to use SSL.

### Obtain Certificate in Standalone Mode

Use when Nginx is stopped or for temporary certificates.

```bash
systemctl stop nginx
certbot certonly --standalone -d example.com -d www.example.com \
  --non-interactive --agree-tos -m admin@example.com
systemctl start nginx
```

### Obtain Certificate with Webroot Mode

Use when Nginx is running but you don't want the plugin to modify config.

```bash
certbot certonly --webroot -w /var/www/html -d example.com -d www.example.com \
  --non-interactive --agree-tos -m admin@example.com
```

### Wildcard Certificate (DNS Challenge)

Requires a DNS plugin. Example with Cloudflare:

```bash
# Create Cloudflare API credentials file
mkdir -p ~/.secrets
cat > ~/.secrets/cloudflare.ini << 'EOF'
dns_cloudflare_api_token = your-api-token-here
EOF
chmod 600 ~/.secrets/cloudflare.ini

# Get wildcard certificate
certbot certonly --dns-cloudflare \
  --dns-cloudflare-credentials ~/.secrets/cloudflare.ini \
  -d example.com -d *.example.com \
  --non-interactive --agree-tos -m admin@example.com
```

Supported DNS plugins: cloudflare, route53, digitalocean, google, linode, ovh, and [many more](https://certbot.eff.org/docs/using.html#dns-plugins).

### Multiple Domains on One Certificate

```bash
certbot --nginx -d example.com -d www.example.com -d blog.example.com -d api.example.com
```

Limit of 100 domain names per certificate.

## Nginx Configuration

### Manual Nginx SSL Block

```nginx
server {
    listen 443 ssl http2;
    server_name example.com www.example.com;

    ssl_certificate /etc/letsencrypt/live/example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem;
    ssl_trusted_certificate /etc/letsencrypt/live/example.com/chain.pem;

    # Strong SSL settings
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384;
    ssl_prefer_server_ciphers on;
    ssl_session_cache shared:SSL:10m;
    ssl_session_timeout 1d;

    # HSTS
    add_header Strict-Transport-Security "max-age=63072000" always;

    root /var/www/html;
    index index.html;
}

# Redirect HTTP to HTTPS
server {
    listen 80;
    server_name example.com www.example.com;
    return 301 https://$server_name$request_uri;
}
```

### HTTP-Only for Let's Encrypt Validation

```nginx
server {
    listen 80;
    server_name example.com www.example.com;

    # Let's Encrypt validation
    location /.well-known/acme-challenge/ {
        root /var/www/html;
    }

    # Redirect everything else to HTTPS
    location / {
        return 301 https://$server_name$request_uri;
    }
}
```

## Auto-Renewal

### Systemd Timer (Default on Most Distros)

```bash
# Check if timer exists
systemctl list-timers | grep certbot

# Enable timer
systemctl enable --now certbot.timer

# View timer status
systemctl status certbot.timer
```

### Cron Job (Manual Setup)

```bash
# Edit crontab
crontab -e

# Renew twice daily (recommended)
0 */12 * * * certbot renew --quiet --post-hook "systemctl reload nginx"

# Or once daily at 3 AM
0 3 * * * certbot renew --quiet --post-hook "systemctl reload nginx"
```

### Systemd Service + Timer (without default certbot.timer)

Create `/etc/systemd/system/certbot-renew.service`:

```ini
[Unit]
Description=Certbot Renewal

[Service]
Type=oneshot
ExecStart=/usr/bin/certbot renew --quiet --post-hook "systemctl reload nginx"
```

Create `/etc/systemd/system/certbot-renew.timer`:

```ini
[Unit]
Description=Run certbot renewal twice daily

[Timer]
OnCalendar=*-*-* 00,12:00:00
RandomizedDelaySec=3600
Persistent=true

[Install]
WantedBy=timers.target
```

Enable:

```bash
systemctl daemon-reload
systemctl enable --now certbot-renew.timer
```

### Docker Auto-Renew

```bash
# Run renewal in Docker
docker run --rm \
  -v /etc/letsencrypt:/etc/letsencrypt \
  -v /var/lib/letsencrypt:/var/lib/letsencrypt \
  certbot/certbot renew --quiet

# Add to crontab
echo "0 */12 * * * docker run --rm -v /etc/letsencrypt:/etc/letsencrypt -v /var/lib/letsencrypt:/var/lib/letsencrypt certbot/certbot renew --quiet --post-hook 'nginx -s reload'" | crontab -
```

### Force Renewal

```bash
# Force renew (even if not expiring soon)
certbot renew --force-renewal

# Renew specific certificate
certbot certonly --force-renewal -d example.com
```

### Dry Run (Test Without Modifying)

```bash
certbot renew --dry-run
```

## Certificate Management

### List All Certificates

```bash
certbot certificates
```

### View Certificate Details

```bash
# Using Certbot
certbot certificates --cert-name example.com

# Using OpenSSL
openssl x509 -in /etc/letsencrypt/live/example.com/fullchain.pem -text -noout

# Check expiry date
openssl x509 -enddate -noout -in /etc/letsencrypt/live/example.com/fullchain.pem
```

### Revoke a Certificate

```bash
certbot revoke --cert-path /etc/letsencrypt/live/example.com/cert.pem

# Revoke and delete
certbot revoke --cert-path /etc/letsencrypt/live/example.com/cert.pem --delete-after-revoke
```

### Delete a Certificate

```bash
certbot delete --cert-name example.com
```

## Certificate Locations

```
/etc/letsencrypt/
├── live/
│   └── example.com/
│       ├── fullchain.pem     # Full certificate chain (recommended for Nginx)
│       ├── cert.pem          # Certificate only
│       ├── chain.pem         # Intermediate chain
│       ├── privkey.pem       # Private key (keep secure!)
│       └── README
├── archive/                   # Previous versions
├── renewal/                   # Renewal configs
└── renewal-hooks/             # Pre/post renewal hooks
```

### Which File to Use

| File | Use For |
|------|---------|
| `fullchain.pem` | Nginx `ssl_certificate` |
| `privkey.pem` | Nginx `ssl_certificate_key` (keep secret, chmod 600) |
| `chain.pem` | Nginx `ssl_trusted_certificate` (OCSP stapling) |
| `cert.pem` | Rarely used alone, only when chain not needed |

## Renewal Hooks

```bash
# Pre-hook (runs before renewal)
mkdir -p /etc/letsencrypt/renewal-hooks/pre
cat > /etc/letsencrypt/renewal-hooks/pre/stop-nginx.sh << 'EOF'
#!/bin/bash
systemctl stop nginx
EOF
chmod +x /etc/letsencrypt/renewal-hooks/pre/stop-nginx.sh

# Post-hook (runs after successful renewal)
mkdir -p /etc/letsencrypt/renewal-hooks/post
cat > /etc/letsencrypt/renewal-hooks/post/reload-nginx.sh << 'EOF'
#!/bin/bash
systemctl start nginx
EOF
chmod +x /etc/letsencrypt/renewal-hooks/post/reload-nginx.sh
```

## Docker Compose with Certbot

```yaml
services:
  nginx:
    image: nginx:alpine
    container_name: nginx
    restart: unless-stopped
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf:ro
      - ./html:/usr/share/nginx/html
      - /etc/letsencrypt:/etc/letsencrypt:ro

  certbot:
    image: certbot/certbot
    container_name: certbot
    entrypoint: "/bin/sh -c 'trap exit TERM; while :; do certbot renew; sleep 12h & wait $${!}; done;'"
    volumes:
      - /etc/letsencrypt:/etc/letsencrypt
      - /var/lib/letsencrypt:/var/lib/letsencrypt
      - ./html:/var/www/html
```

Initial certificate with Docker:

```bash
docker compose run --rm certbot certonly --webroot \
  -w /var/www/html -d example.com -d www.example.com \
  --agree-tos -m admin@example.com

docker compose restart nginx
```

## ECC Certificates (Elliptic Curve)

```bash
# Generate ECDSA key instead of RSA
certbot certonly --nginx -d example.com \
  --key-type ecdsa --curve secp256r1

# Set default key type for all future certificates
echo "key-type = ecdsa" >> /etc/letsencrypt/cli.ini
```

## ocsp Stapling (Nginx)

```nginx
ssl_stapling on;
ssl_stapling_verify on;
ssl_trusted_certificate /etc/letsencrypt/live/example.com/chain.pem;
resolver 1.1.1.1 8.8.8.8 valid=300s;
resolver_timeout 5s;
```

## Troubleshooting

### Check Logs

```bash
journalctl -u certbot -n 50
tail -f /var/log/letsencrypt/letsencrypt.log
```

### Rate Limits

Let's Encrypt has these limits:

| Limit | Per |
|-------|-----|
| 50 certificates/week | Registered domain |
| 5 certificates/week | Duplicate domain set |
| 300 orders/3 hours | Account |

```bash
# Check rate limit status
curl -s https://letsencrypt.org/ | grep -i rate

# List issued certificates for a domain
certbot certificates --cert-name example.com | grep "Domains:"
```

### Port 80 Already in Use

```bash
# Stop whatever is using port 80
fuser -k 80/tcp

# Or use webroot mode instead of standalone
certbot certonly --webroot -w /var/www/html -d example.com
```

### Certificate Not Trusted

```bash
# Ensure fullchain.pem is used (not cert.pem alone)
ssl_certificate /etc/letsencrypt/live/example.com/fullchain.pem;
```

### Renewal Failed

```bash
# Test renewal manually
certbot renew --verbose

# Check DNS propagation
dig +short example.com

# Ensure port 80 is accessible
curl -I http://example.com/.well-known/acme-challenge/test
```

## Complete Automation Script

```bash
#!/bin/bash
# init-ssl.sh — First-time SSL setup for Nginx

DOMAIN="${1:-example.com}"
EMAIL="${2:-admin@example.com}"

if [ -d "/etc/letsencrypt/live/$DOMAIN" ]; then
    echo "Certificate already exists for $DOMAIN"
    exit 0
fi

# Install Certbot
if command -v apt &>/dev/null; then
    apt update && apt install -y certbot python3-certbot-nginx
elif command -v dnf &>/dev/null; then
    dnf install -y epel-release && dnf install -y certbot python3-certbot-nginx
fi

# Obtain certificate
certbot --nginx -d "$DOMAIN" -d "www.$DOMAIN" \
    --non-interactive --agree-tos -m "$EMAIL"

# Set up auto-renewal
echo "0 */12 * * * root certbot renew --quiet --post-hook \"systemctl reload nginx\"" > /etc/cron.d/certbot-renew

# Test renewal
certbot renew --dry-run && echo "SSL setup complete for $DOMAIN"
```

## Quick Reference

```bash
# Install
apt install certbot python3-certbot-nginx

# Get cert (Nginx)
certbot --nginx -d example.com

# Get cert (standalone)
certbot certonly --standalone -d example.com

# Get wildcard cert (DNS)
certbot certonly --dns-cloudflare -d example.com -d *.example.com

# Renew all
certbot renew

# Renew with reload
certbot renew --post-hook "systemctl reload nginx"

# Force renew
certbot renew --force-renewal

# List certs
certbot certificates

# Delete cert
certbot delete --cert-name example.com

# Dry run renewal
certbot renew --dry-run

# Check expiry
openssl x509 -enddate -noout -in /etc/letsencrypt/live/example.com/fullchain.pem
```
