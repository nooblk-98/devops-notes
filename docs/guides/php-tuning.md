# PHP Tuning

## PHP-FPM Pool Configuration

```ini title="/etc/php/8.2/fpm/pool.d/www.conf"
[www]
user = www-data
group = www-data

listen = 127.0.0.1:9000
# OR listen = /run/php/php8.2-fpm.sock

pm = dynamic
pm.max_children = 50
pm.start_servers = 5
pm.min_spare_servers = 5
pm.max_spare_servers = 35
pm.max_requests = 500
```

### pm.max_children Calculation

```
Available RAM = Server RAM - OS + other services
Each PHP process ≈ 30-50MB

Example: 4GB server, 1GB for OS/services
Available = 3GB = 3072MB
max_children = 3072 / 50 ≈ 60
```

### Pool Settings by Traffic

| Traffic | pm | max_children | start_servers |
|---------|----|-------------|---------------|
| Low | dynamic | 10 | 2 |
| Medium | dynamic | 50 | 5 |
| High | static | 100 | 100 |
| WordPress | dynamic | 30-80 | 5 |

## OPcache Configuration

```ini title="/etc/php/8.2/cli/conf.d/10-opcache.ini"
opcache.enable=1
opcache.memory_consumption=256
opcache.interned_strings_buffer=16
opcache.max_accelerated_files=10000
opcache.revalidate_freq=2
opcache.fast_shutdown=1
opcache.validate_timestamps=1
opcache.revalidate_path=0
opcache.save_comments=1
```

## PHP.INI Settings

```ini title="/etc/php/8.2/fpm/php.ini"
memory_limit = 256M
upload_max_filesize = 64M
post_max_size = 64M
max_execution_time = 300
max_input_time = 300
max_input_vars = 3000
date.timezone = Asia/Colombo

; Security
disable_functions = exec,passthru,shell_exec,system,proc_open,popen,curl_exec,curl_multi_exec,parse_ini_file,show_source
expose_php = Off
display_errors = Off
log_errors = On
error_log = /var/log/php_errors.log

; Performance
realpath_cache_size = 4096K
realpath_cache_ttl = 600
```

## WordPress-Specific Tuning

```ini title="/etc/php/8.2/fpm/conf.d/90-wordpress.ini"
; WordPress often needs more
memory_limit = 256M
upload_max_filesize = 128M
post_max_size = 128M
max_execution_time = 180
```

## Per-Site Pool (Multi-Site)

Create separate pools per site:

```ini title="/etc/php/8.2/fpm/pool.d/site1.conf"
[site1]
user = site1
group = site1
listen = 127.0.0.1:9001
pm = dynamic
pm.max_children = 20
pm.start_servers = 3
pm.min_spare_servers = 2
pm.max_spare_servers = 10
chdir = /var/www/site1
```

```ini title="/etc/php/8.2/fpm/pool.d/site2.conf"
[site2]
user = site2
group = site2
listen = 127.0.0.1:9002
pm = dynamic
pm.max_children = 30
pm.start_servers = 5
pm.min_spare_servers = 3
pm.max_spare_servers = 15
chdir = /var/www/site2
```

## Nginx + PHP-FPM

```nginx
location ~ \.php$ {
    fastcgi_pass 127.0.0.1:9001;  # per-site pool
    fastcgi_index index.php;
    fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
    include fastcgi_params;

    # Buffer settings
    fastcgi_buffers 16 16k;
    fastcgi_buffer_size 32k;
    fastcgi_busy_buffers_size 256k;
}
```

## Monitoring PHP-FPM

```bash
# Check status
systemctl status php8.2-fpm

# Check pool statistics
curl http://localhost:9000/status
# Need status_path set in pool config

# Process count
ps aux | grep php-fpm | wc -l

# Memory per process
ps -ylC php-fpm --sort:rss
```

## Restart After Changes

```bash
php-fpm8.2 -t && systemctl reload php8.2-fpm
```

## Verification

- [ ] `pm.max_children` matches server RAM
- [ ] OPcache enabled and memory adequate
- [ ] Upload limits match site requirements
- [ ] `expose_php = Off`
- [ ] `display_errors = Off` in production
- [ ] PHP-FPM status page accessible (if enabled)
- [ ] Separate pools for multi-site setups
