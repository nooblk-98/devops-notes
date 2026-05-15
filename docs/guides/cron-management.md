# Cron Job Management

## Crontab Basics

```bash
# Edit crontab for current user
crontab -e

# List cron jobs
crontab -l

# Edit for specific user
crontab -u www-data -e

# Remove all cron jobs
crontab -r
```

### Crontab Format

```
* * * * * command
│ │ │ │ │
│ │ │ │ └── Day of week (0-7, 0=Sun)
│ │ │ └──── Month (1-12)
│ │ └────── Day of month (1-31)
│ └──────── Hour (0-23)
└────────── Minute (0-59)
```

### Examples

```bash
# Every minute
* * * * * /usr/bin/php /var/www/html/cron.php

# Every day at 3 AM
0 3 * * * /usr/bin/php /var/www/html/daily.php

# Every Monday at 2 AM
0 2 * * 1 /usr/bin/php /var/www/html/weekly.php

# Every hour
0 * * * * /usr/bin/php /var/www/html/hourly.php

# Every 15 minutes
*/15 * * * * /usr/bin/php /var/www/html/quarterly.php

# First day of month at midnight
0 0 1 * * /usr/bin/php /var/www/html/monthly.php
```

## Common WordPress Cron Jobs

### WP-Cron Replacement (System Cron)

Disable WP-Cron and use system cron:

```php title="wp-config.php"
define('DISABLE_WP_CRON', true);
```

```bash
# Add to crontab
* * * * * wget -q -O - https://example.com/wp-cron.php?doing_wp_cron=1 >/dev/null 2>&1

# Or via CLI (requires wp-cli)
* * * * * /usr/local/bin/wp cron event run --due-now --path=/var/www/html >> /var/log/wp-cron.log 2>&1
```

### Common WordPress Jobs

```bash
# Run WordPress cron every 5 minutes
*/5 * * * * /usr/local/bin/wp cron event run --due-now --path=/var/www/html >> /var/log/wp-cron.log 2>&1

# WordPress core updates (weekly)
0 3 * * 0 /usr/local/bin/wp core update --path=/var/www/html >> /var/log/wp-update.log 2>&1

# Plugin updates (weekly)
0 4 * * 0 /usr/local/bin/wp plugin update --all --path=/var/www/html >> /var/log/wp-update.log 2>&1

# Database optimize (monthly)
0 5 1 * * /usr/local/bin/wp db optimize --path=/var/www/html >> /var/log/wp-maintenance.log 2>&1

# Search and replace (if needed)
0 6 * * 1 /usr/local/bin/wp search-replace 'http://' 'https://' --path=/var/www/html --dry-run >> /var/log/wp-search.log 2>&1
```

## System Maintenance Cron Jobs

```bash
# Daily database backup
0 2 * * * /usr/bin/mysqldump -u root -p$(cat /etc/mysql/backup-pass) wordpress | gzip > /backups/db/wordpress-$(date +\%Y\%m\%d).sql.gz

# Weekly full backup
0 3 * * 0 tar czf /backups/full/wordpress-full-$(date +\%Y\%m\%d).tar.gz /var/www/html

# Certificate renewal check
0 0 * * * /usr/bin/certbot renew --quiet --no-self-upgrade

# System updates
0 5 * * 1 /usr/bin/apt update && /usr/bin/apt upgrade -y >> /var/log/apt-upgrade.log 2>&1

# Docker cleanup
0 4 * * 0 /usr/bin/docker system prune -af --volumes >> /var/log/docker-cleanup.log 2>&1

# Logrotate force
0 0 * * * /usr/sbin/logrotate -f /etc/logrotate.conf >> /var/log/logrotate.log 2>&1

# Disk usage alert
0 * * * * /bin/bash -c 'df -h | awk '\''$5 > 85 {print $0}'\'' | mail -s "Disk Alert" admin@example.com'

# Restart services (if hanging)
*/30 * * * * /bin/systemctl is-active nginx || /bin/systemctl restart nginx
```

## Cron Job Logging

```bash
# Log to file
0 3 * * * /usr/bin/php /script.php >> /var/log/cron.log 2>&1

# Log to syslog
0 3 * * * /usr/bin/php /script.php 2>&1 | logger -t mycron

# No logging
0 3 * * * /usr/bin/php /script.php >/dev/null 2>&1
```

## Monitor Cron Execution

```bash
# Check cron logs
grep -i "cron" /var/log/syslog
journalctl -u cron -n 50

# Check failed cron jobs
grep -i "error\|fail" /var/log/cron.log

# Check if cron daemon is running
systemctl status cron
```

## Troubleshooting

| Symptom | Cause | Fix |
|---------|-------|-----|
| Cron not running | Cron daemon stopped | `systemctl enable --now cron` |
| Script not executing | Wrong path | Always use full paths |
| Permission denied | Wrong user | Use correct user: `crontab -u www-data -e` |
| Email flood | No output redirect | Add `>/dev/null 2>&1` |
| WP-Cron not firing | WP-Cron disabled, no replacement | Add system cron for wp-cron.php |

## Best Practices

- Always use full paths in cron commands
- Redirect output to log files (never rely on mail)
- Set PATH at top of crontab: `PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin`
- Use `flock` to prevent overlapping jobs: `*/5 * * * * /usr/bin/flock -n /tmp/lockfile /usr/bin/php /script.php`
- Test commands manually first before adding to crontab
- Document all cron jobs
- Monitor cron logs regularly
