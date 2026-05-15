---
tags:
  - migration
  - server-migration
  - database-migration
---

# Server & Site Migration SOP

**Version:** 1.0  
**Owner:** DevOps Team

## Purpose

Standardize the process of migrating websites and services between servers or hosting providers.

## Scope

Full server migration including files, databases, DNS, and SSL certificates.

## Prerequisites

- SSH access to both old and new servers
- Root/sudo access on both servers
- DNS management access
- Maintenance window scheduled

## Procedure

### 1. Pre-Migration Checklist

- [ ] New server provisioned and hardened
- [ ] Same software versions installed on both servers
- [ ] Enough disk space on new server
- [ ] Maintenance page prepared
- [ ] Rollback plan documented
- [ ] Stakeholders notified

### 2. Backup Old Server

```bash
# Backup files
tar czf /tmp/site-backup-$(date +%Y%m%d).tar.gz /var/www/html/

# Backup MySQL database
mysqldump -u root -p --all-databases > /tmp/db-backup-$(date +%Y%m%d).sql

# Backup PostgreSQL
pg_dumpall -U postgres > /tmp/pg-backup-$(date +%Y%m%d).sql

# Backup configs
tar czf /tmp/config-backup-$(date +%Y%m%d).tar.gz \
  /etc/nginx/ \
  /etc/php/ \
  /etc/mysql/ \
  /etc/letsencrypt/
```

### 3. Transfer to New Server

```bash
# From old server, transfer to new server
rsync -avz --progress -e ssh /tmp/site-backup-$(date +%Y%m%d).tar.gz user@new-server:/tmp/
rsync -avz --progress -e ssh /tmp/db-backup-$(date +%Y%m%d).sql user@new-server:/tmp/
rsync -avz --progress -e ssh /tmp/config-backup-$(date +%Y%m%d).tar.gz user@new-server:/tmp/
```

### 4. Restore on New Server

```bash
# Extract files
tar xzf /tmp/site-backup-$(date +%Y%m%d).tar.gz -C /
chown -R www-data:www-data /var/www/html/

# Restore database
mysql -u root -p < /tmp/db-backup-$(date +%Y%m%d).sql

# Restore configs
tar xzf /tmp/config-backup-$(date +%Y%m%d).tar.gz -C /
```

### 5. Update Configs for New Server

```bash
# Update database host if changed
# Update file paths in configs
# Update .env with new server values

# Test Nginx config
nginx -t && systemctl reload nginx
```

### 6. Set Up SSL

```bash
# Generate new certificate if new domain
certbot --nginx -d example.com

# Or copy old certs
cp /tmp/letsencrypt/* /etc/letsencrypt/
```

### 7. Test on New Server

```bash
# Test locally (edit /etc/hosts first)
curl -I http://localhost
curl -I https://localhost

# Check database connections
php -r "new PDO('mysql:host=localhost;dbname=wordpress', 'user', 'pass'); echo 'OK\n';"
```

### 8. Switch DNS

Update DNS records to point to new server IP:

| Record | Type | Value | TTL |
|--------|------|-------|-----|
| example.com | A | <new-ip> | 300 |
| www.example.com | CNAME | example.com | 300 |

Set TTL low (300s) before migration, restore after.

### 9. Post-Migration Verification

```bash
# Verify DNS propagated
nslookup example.com
dig example.com +short

# Verify site works
curl -I https://example.com

# Check SSL
curl -vI https://example.com 2>&1 | grep "SSL certificate"

# Monitor error logs
tail -f /var/log/nginx/error.log
```

## Rollback Plan

```bash
# Point DNS back to old server IP
# Verify old server is still running
# Restore old DNS TTL values
# Notify stakeholders
```

## Database Migration Only

```bash
# Export from old
mysqldump -h old-host -u user -p database > data.sql

# Import to new
mysql -h new-host -u user -p database < data.sql

# Update wp-config or app config with new DB host
```

## WordPress-Specific

```bash
# Update URLs after migration
wp search-replace "old-domain.com" "new-domain.com"

# Update wp-config with new DB credentials
# Flush permalinks
wp rewrite flush
```

## Verification

- [ ] All files transferred and permissions correct
- [ ] Database imported and connections working
- [ ] SSL certificate valid
- [ ] DNS propagated to new server
- [ ] Site loads correctly on new server
- [ ] Forms, logins, and critical paths working
- [ ] Old server still available for rollback
- [ ] Monitoring alerts configured for new server
