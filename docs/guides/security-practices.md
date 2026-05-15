# WordPress Security Practices

## 1. File Permissions

```bash
# Set correct ownership
chown -R www-data:www-data /var/www/html

# Directories: 755
find /var/www/html -type d -exec chmod 755 {} \;

# Files: 644
find /var/www/html -type f -exec chmod 644 {} \;

# wp-config.php: 600 (more restrictive)
chmod 600 /var/www/html/wp-config.php
```

## 2. Harden wp-config.php

```php
// Disable file editing in admin
define('DISALLOW_FILE_EDIT', true);

// Limit post revisions
define('WP_POST_REVISIONS', 5);

// Force SSL
define('FORCE_SSL_ADMIN', true);

// Set keys and salts (use https://api.wordpress.org/secret-key/1.1/salt/)
// Add unique AUTH_KEY, SECURE_AUTH_KEY, LOGGED_IN_KEY, NONCE_KEY, etc.
```

## 3. Disable XML-RPC

Block `xmlrpc.php` in Nginx:

```nginx
location /xmlrpc.php {
    deny all;
    return 403;
}
```

Or use a plugin or `.htaccess`.

## 4. Nginx Security Headers

```nginx
add_header X-Frame-Options "SAMEORIGIN" always;
add_header X-Content-Type-Options "nosniff" always;
add_header X-XSS-Protection "1; mode=block" always;
add_header Referrer-Policy "strict-origin-when-cross-origin" always;
add_header Permissions-Policy "geolocation=(), microphone=(), camera=()" always;
```

## 5. Login Hardening

```nginx
# Rate limit wp-login.php
location /wp-login.php {
    limit_req zone=login burst=5 nodelay;
    fastcgi_pass 127.0.0.1:9000;
    include fastcgi_params;
    fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
}
```

Define the zone in the `http` block:

```nginx
limit_req_zone $binary_remote_addr zone=login:10m rate=3r/m;
```

## 6. Nginx Security Headers

Add to the `server` block in `/etc/nginx/sites-available/example.com`:

```nginx
server {
    # ...
    
    # HSTS — force HTTPS (1 year)
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains; preload" always;

    # Prevent MIME type sniffing
    add_header X-Content-Type-Options "nosniff" always;

    # Prevent clickjacking
    add_header X-Frame-Options "SAMEORIGIN" always;

    # XSS protection (legacy browsers)
    add_header X-XSS-Protection "1; mode=block" always;

    # Referrer policy
    add_header Referrer-Policy "strict-origin-when-cross-origin" always;

    # Permissions Policy (restrict browser features)
    add_header Permissions-Policy "geolocation=(), microphone=(), camera=(), payment=(), usb=(), fullscreen=(self)" always;

    # Content Security Policy
    add_header Content-Security-Policy "
        default-src 'self';
        script-src 'self' 'unsafe-inline' https://www.googletagmanager.com;
        style-src 'self' 'unsafe-inline' https://fonts.googleapis.com;
        img-src 'self' data: https:;
        font-src 'self' https://fonts.gstatic.com;
        connect-src 'self';
        frame-src 'none';
        object-src 'none';
        upgrade-insecure-requests;
    " always;
}
```

### Apply

```bash
nginx -t && systemctl reload nginx
```

### Verify Headers

```bash
curl -I https://example.com | grep -i "^add-header\|^strict\|^x-\|^content-security\|^referrer"
```

Expected output:

```
strict-transport-security: max-age=31536000; includeSubDomains; preload
x-content-type-options: nosniff
x-frame-options: SAMEORIGIN
x-xss-protection: 1; mode=block
referrer-policy: strict-origin-when-cross-origin
permissions-policy: geolocation=(), microphone=(), camera=(), payment=(), usb=(), fullscreen=(self)
content-security-policy: default-src 'self'; script-src 'self' ...
```

| Header | Purpose |
|--------|---------|
| `Strict-Transport-Security` | Forces HTTPS for 1 year, including subdomains |
| `X-Content-Type-Options` | Prevents MIME sniffing attacks |
| `X-Frame-Options` | Prevents clickjacking |
| `X-XSS-Protection` | Enables browser XSS filter |
| `Referrer-Policy` | Controls referrer header sent with requests |
| `Permissions-Policy` | Restricts which browser APIs pages can use |
| `Content-Security-Policy` | Prevents XSS and data injection attacks |

## 7. Database Security

```bash
# Use separate database user per WordPress site (not root)
# Strong password (16+ chars with special characters)

# Remove default test database
mysql -e "DROP DATABASE IF EXISTS test;"

# Remove anonymous users
mysql -e "DELETE FROM mysql.user WHERE User='';"

# Disable remote root login
mysql -e "DELETE FROM mysql.user WHERE User='root' AND Host NOT IN ('localhost', '127.0.0.1', '::1');"

# Apply changes
mysql -e "FLUSH PRIVILEGES;"
```

## 8. Regular Updates

```bash
# Core, themes, plugins via WP-CLI
wp core update
wp plugin update --all
wp theme update --all
```

## 9. Plugins (Recommended)

- **WordFence** — Firewall and malware scanner
- **WPS Hide Login** — Change login URL from `/wp-admin`
- **Limit Login Attempts Reloaded** — Brute force protection
- **UpdraftPlus** — Automated backups
- **Really Simple SSL** — SSL configuration helper

## 10. Server-Level Security

```bash
# Disable root SSH login
sed -i 's/PermitRootLogin yes/PermitRootLogin no/' /etc/ssh/sshd_config
systemctl reload sshd

# Use SSH key auth only
sed -i 's/PasswordAuthentication yes/PasswordAuthentication no/' /etc/ssh/sshd_config

# Fail2ban for WordPress
apt install -y fail2ban
cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local

# WordPress-specific fail2ban filter
cat > /etc/fail2ban/jail.d/wordpress.conf << 'EOF'
[wordpress]
enabled = true
port = http,https
filter = wordpress
logpath = /var/log/nginx/access.log
maxretry = 5
bantime = 3600
EOF
```

## 11. Monitoring

- Set up daily malware scans
- Monitor file integrity (check for changes to core files)
- Audit user accounts monthly
- Review error logs weekly

## Checklist

- [ ] File permissions set correctly
- [ ] File editing disabled in wp-config
- [ ] XML-RPC blocked
- [ ] SSL forced
- [ ] Security headers added
- [ ] Login rate limited
- [ ] Database hardened
- [ ] Root SSH login disabled
- [ ] Fail2ban configured
- [ ] Backups automated
- [ ] Updates regularly applied
