---
tags:
  - wordpress
  - migration
  - domain
  - url
---

# WordPress Domain Change

**Version:** 1.0  
**Last Updated:** 2024-01-01  
**Owner:** DevOps Team

## Purpose

Change a WordPress site's URL from one domain to another (e.g., `example.com` to `example.lk`).

## Scope

This SOP covers three methods for changing the Site URL and Home URL in WordPress.

## Prerequisites

- SSH or shell access to the server
- Database credentials (or `wp-cli` installed)
- New domain DNS pointed to server before proceeding
- Full database backup taken before making changes

## WP-CLI Installation

If `wp` is not available on the server, install WP-CLI:

```bash
# Download WP-CLI
curl -O https://raw.githubusercontent.com/wp-cli/builds/gh-pages/phar/wp-cli.phar

# Verify it works
php wp-cli.phar --info

# Make executable and move to PATH
chmod +x wp-cli.phar
sudo mv wp-cli.phar /usr/local/bin/wp

# Verify installation
wp --version
```

If WP-CLI is inside a Docker container (Docker-based setup):

```bash
# Install WP-CLI inside the WordPress container
docker compose exec wordpress curl -O https://raw.githubusercontent.com/wp-cli/builds/gh-pages/phar/wp-cli.phar
docker compose exec wordpress chmod +x wp-cli.phar
docker compose exec wordpress mv wp-cli.phar /usr/local/bin/wp

# Verify
docker compose exec wordpress wp --version

# Use it directly
docker compose exec wordpress wp search-replace "example.com" "example.lk"
```

## Procedure

### 0. Pre-Migration Backup

```bash
# Backup database
docker compose exec maria-db mysqldump -u wpuser -p wordpress > wp-backup-before-url-change.sql

# Backup files
tar czf wp-files-backup-before-url-change.tar.gz html/
```

### Method 1: wp-cli (Recommended)

```bash
# Run wp-cli on the WordPress container
docker compose exec wordpress wp search-replace "example.com" "example.lk" --skip-columns=guid

# Verify
docker compose exec wordpress wp option get siteurl
docker compose exec wordpress wp option get home
```

The `search-replace` command updates all occurrences in the database including post content, options, widgets, and serialized data.

### Method 2: Direct Database Update (phpMyAdmin / SQL)

```bash
# Connect to database
docker compose exec maria-db mysql -u wpuser -p wordpress
```

```sql
-- Update site URL and home URL
UPDATE wp_options SET option_value = 'https://example.lk' WHERE option_name = 'siteurl';
UPDATE wp_options SET option_value = 'https://example.lk' WHERE option_name = 'home';

-- Update content references (run for each table)
UPDATE wp_posts SET guid = REPLACE(guid, 'example.com', 'example.lk');
UPDATE wp_posts SET post_content = REPLACE(post_content, 'example.com', 'example.lk');
UPDATE wp_postmeta SET meta_value = REPLACE(meta_value, 'example.com', 'example.lk');
UPDATE wp_options SET option_value = REPLACE(option_value, 'example.com', 'example.lk');
```

!!! warning "Serialized Data"
    WordPress stores some options as serialized PHP arrays. Using raw SQL `REPLACE` on serialized data can corrupt string lengths. Always use Method 1 (wp-cli) or Method 4 (intervention script) for safety.

### Method 3: wp-config.php Override (Quick Workaround)

```php title="html/wp-config.php"
// Add above "That's all, stop editing!" line
define('WP_HOME', 'https://example.lk');
define('WP_SITEURL', 'https://example.lk');

// Force HTTPS and domain in admin
define('FORCE_SSL_ADMIN', true);
```

This method doesn't update the database — it overrides the URL at the code level. Useful for quickly accessing the admin before running `search-replace`.

### Method 4: Safe Serialize-Aware Script

Create a PHP script for environments without wp-cli:

```php title="fix-domain.php"
<?php
require_once('wp-config.php');
require_once('wp-includes/functions.php');

$old = 'example.com';
$new = 'example.lk';

$tables = ['wp_posts', 'wp_postmeta', 'wp_options', 'wp_comments', 'wp_terms'];
foreach ($tables as $table) {
    $rows = $wpdb->get_results("SELECT * FROM $table");
    foreach ($rows as $row) {
        foreach ($row as $col => $val) {
            if (is_string($val) && strpos($val, $old) !== false) {
                $fixed = str_replace($old, $new, $val);
                $wpdb->update($table, [$col => $fixed], ['ID' => $row->ID]);
            }
        }
    }
}
echo "Domain URLs updated.\n";
```

Remove this file after running.

### 5. Post-Migration Tasks

```bash
# Flush permalinks (visit Settings → Permalinks → Save)
docker compose exec wordpress wp rewrite flush

# Clear cache (if using caching plugin)
docker compose exec wordpress wp cache flush

# Update Nginx config if domain is in server_name
```

### 6. Update Nginx Configuration

```nginx title="nginx/nginx.conf"
server_name example.lk;  # was example.com
```

```bash
docker compose exec nginx nginx -s reload
```

### 7. Verification

```bash
# Check site responds on new domain
curl -I https://example.lk

# Verify old domain redirects (if needed)
curl -I https://example.com

# Check internal links are using new domain
docker compose exec wordpress wp option get siteurl
docker compose exec wordpress wp option get home

# Login to admin on new domain
# https://example.lk/wp-admin
```

## Rollback

If the new domain has issues, restore the database backup:

```bash
docker compose exec -T maria-db mysql -u wpuser -p wordpress < wp-backup-before-url-change.sql
```

Then revert any Nginx config changes, restore file backup if needed, and point DNS back to old provider.

## Troubleshooting

| Symptom | Cause | Fix |
|---------|-------|-----|
| Redirect loop | Cached redirect in browser | Clear browser cache / incognito |
| Admin login redirects to old domain | `siteurl` not updated | Use wp-config.php override (Method 3) |
| Broken images | GUIDs not updated | Re-run search-replace on wp_posts |
| "Database connection error" | Wrong credentials | Verify `.env` and restart stack |

## Verification

- [ ] Database backed up before migration
- [ ] `siteurl` and `home` options updated
- [ ] Post content URLs updated to new domain
- [ ] Permalinks flushed
- [ ] Nginx config updated
- [ ] DNS for new domain resolves
- [ ] Site loads on new domain
- [ ] Admin login works on new domain
- [ ] Old domain redirects or is decommissioned
