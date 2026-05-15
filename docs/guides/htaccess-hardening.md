# Hardening WordPress with .htaccess

!!! note "Nginx Users"
    `.htaccess` is Apache-only. If using Nginx, use `server` block directives instead.

## Block Access to Sensitive Files

```apache title=".htaccess (in WordPress root)"
# Block wp-config.php
<Files wp-config.php>
    order allow,deny
    deny from all
</Files>

# Block access to .htaccess itself
<Files .htaccess>
    order allow,deny
    deny from all
</Files>

# Block access to .git
RedirectMatch 404 /\.git

# Block readme / license files
<Files ~ "(readme|license|changelog|README)\.(html|txt|md)$">
    order allow,deny
    deny from all
</Files>

# Block XML-RPC (if not needed)
<Files xmlrpc.php>
    order allow,deny
    deny from all
</Files>

# Block wp-includes version.php
<Files version.php>
    order allow,deny
    deny from all
</Files>
```

## Block PHP Execution in Uploads

```apache
# Prevent PHP execution in uploads folder
<Directory "/var/www/html/wp-content/uploads">
    <Files *.php>
        deny from all
    </Files>
</Directory>
```

For Nginx:

```nginx
location /wp-content/uploads/ {
    location ~ \.php$ { deny all; }
}
```

## Deny Access to wp-content (with exceptions)

```apache
# Deny access to wp-content but allow specific file types
<Directory "/var/www/html/wp-content">
    <FilesMatch "\.(php|php\.)$">
        deny from all
    </FilesMatch>
</Directory>
```

## IP Blocking

```apache
# Block specific IPs
order allow,deny
deny from 192.168.1.100
deny from 10.0.0.0/24
allow from all

# Block by country (requires GeoIP module)
# Allow only specific countries
GeoIPEnable On
SetEnvIf GEOIP_COUNTRY_CODE US AllowCountry
SetEnvIf GEOIP_COUNTRY_CODE GB AllowCountry
order deny,allow
deny from all
allow from env=AllowCountry

# Block known bad bots
SetEnvIfNoCase User-Agent "^AhrefsBot" bad_bot
SetEnvIfNoCase User-Agent "^SemrushBot" bad_bot
SetEnvIfNoCase User-Agent "^MJ12bot" bad_bot
deny from env=bad_bot
```

## Protect wp-admin

```apache
# Additional password protect wp-admin
<Directory "/var/www/html/wp-admin">
    AuthType Basic
    AuthName "Restricted Access"
    AuthUserFile /etc/apache2/.htpasswd
    Require valid-user
</Directory>
```

```bash
htpasswd -c /etc/apache2/.htpasswd admin
```

### IP Restrict wp-admin

```apache
# Only allow office IPs
order deny,allow
deny from all
allow from 203.0.113.0/24  # office IP range
allow from 198.51.100.50    # VPN IP
```

## Hotlink Protection

```apache
# Prevent other sites from embedding your images
RewriteEngine on
RewriteCond %{HTTP_REFERER} !^$
RewriteCond %{HTTP_REFERER} !^https://(www\.)?example\.com [NC]
RewriteCond %{HTTP_REFERER} !^https://(www\.)?google\.com [NC]
RewriteRule \.(jpg|jpeg|png|gif|webp|svg)$ - [F]
```

## Force HTTPS

```apache
RewriteEngine On
RewriteCond %{HTTPS} off
RewriteRule ^(.*)$ https://%{HTTP_HOST}/$1 [R=301,L]
```

## Security Headers

```apache
<IfModule mod_headers.c>
    Header set X-Frame-Options "SAMEORIGIN"
    Header set X-Content-Type-Options "nosniff"
    Header set X-XSS-Protection "1; mode=block"
    Header set Referrer-Policy "strict-origin-when-cross-origin"
    Header set Permissions-Policy "geolocation=(), microphone=(), camera=()"
    Header always set Strict-Transport-Security "max-age=31536000; includeSubDomains"
</IfModule>
```

## Block URL-Based Attacks

```apache
# Block suspicious query strings
RewriteCond %{QUERY_STRING} (eval|base64|union|select|insert|drop) [NC,OR]
RewriteCond %{QUERY_STRING} (mosConfig|phpinfo|\(\)|shell_exec) [NC]
RewriteRule .* - [F]

# Block suspicious user agents
RewriteCond %{HTTP_USER_AGENT} (libwww|wget|nikto|curl|scan|python|perl) [NC]
RewriteRule .* - [F]

# Block malicious requests to wp-includes
RewriteRule ^wp-includes/(.*)\.php$ - [F]

# Block direct access to PHP files in wp-content
RewriteRule ^wp-content/.*\.php$ - [F]
```

## Complete Hardened .htaccess

```apache
# BEGIN Hardened WordPress
<IfModule mod_rewrite.c>
    RewriteEngine On

    # Force HTTPS
    RewriteCond %{HTTPS} off
    RewriteRule ^(.*)$ https://%{HTTP_HOST}/$1 [R=301,L]

    # Block bad query strings
    RewriteCond %{QUERY_STRING} (eval|base64|union|select|insert|drop) [NC,OR]
    RewriteCond %{QUERY_STRING} (mosConfig|phpinfo|\(\)|shell_exec) [NC]
    RewriteRule .* - [F]

    # Block bad user agents
    RewriteCond %{HTTP_USER_AGENT} (libwww|wget|nikto|curl|scan|python|perl|AhrefsBot|SemrushBot|MJ12bot) [NC]
    RewriteRule .* - [F]
</IfModule>

<IfModule mod_headers.c>
    Header set X-Frame-Options "SAMEORIGIN"
    Header set X-Content-Type-Options "nosniff"
    Header set X-XSS-Protection "1; mode=block"
    Header set Referrer-Policy "strict-origin-when-cross-origin"
    Header set Permissions-Policy "geolocation=(), microphone=(), camera=()"
</IfModule>

# Protect sensitive files
<FilesMatch "^(wp-config\.php|\.htaccess|readme\.html|license\.txt|xmlrpc\.php)$">
    order allow,deny
    deny from all
</FilesMatch>

# Block PHP in uploads
<Directory "/var/www/html/wp-content/uploads">
    <Files *.php>
        deny from all
    </Files>
</Directory>

# WordPress rewrite rules (keep existing)
# BEGIN WordPress
RewriteRule ^index\.php$ - [L]
RewriteCond %{REQUEST_FILENAME} !-f
RewriteCond %{REQUEST_FILENAME} !-d
RewriteRule . /index.php [L]
# END WordPress
# END Hardened WordPress
```

## Test .htaccess

```bash
# Check syntax
apachectl -t

# Reload Apache
systemctl reload apache2

# Test blocks
curl -I https://example.com/wp-config.php   # Should return 403
curl -I https://example.com/xmlrpc.php       # Should return 403
curl -I https://example.com/.git/config      # Should return 404
```

## Verification

- [ ] wp-config.php returns 403
- [ ] xmlrpc.php blocked
- [ ] .git directory returns 404
- [ ] Uploads cannot execute PHP
- [ ] HTTPS forced
- [ ] Security headers present
- [ ] Bad user agents blocked
- [ ] Suspicious query strings blocked
