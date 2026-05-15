# Redis & Memcached for WordPress

## Install Redis

```bash
apt install -y redis-server
systemctl enable --now redis
```

### Redis Config

```ini title="/etc/redis/redis.conf"
bind 127.0.0.1
port 6379
maxmemory 256mb
maxmemory-policy allkeys-lru
save 900 1
save 300 10
save 60 10000
```

## Redis as WordPress Object Cache

### Install Redis PHP Extension

```bash
apt install -y php8.2-redis
systemctl reload php8.2-fpm
```

### Install Redis Object Cache Plugin

```bash
# Via WP-CLI
wp plugin install redis-cache --activate

# Enable Redis cache
wp redis enable

# Verify
wp redis status
```

Or manually: Download and copy `redis-cache` plugin to `wp-content/plugins/`, then activate.

### Add to wp-config.php

```php
define('WP_REDIS_HOST', '127.0.0.1');
define('WP_REDIS_PORT', 6379);
define('WP_REDIS_DATABASE', 0);
define('WP_REDIS_TIMEOUT', 1);
define('WP_REDIS_READ_TIMEOUT', 1);
```

### Verify Caching

```bash
# Check Redis hits/misses
redis-cli info stats | grep keyspace

# Monitor live
redis-cli monitor

# Check WordPress cache stats
wp redis info
```

## Install Memcached

```bash
apt install -y memcached php8.2-memcache
systemctl enable --now memcached
```

### Memcached Config

```ini title="/etc/memcached.conf"
-d
logfile /var/log/memcached.log
-p 11211
-l 127.0.0.1
-m 256
-c 1024
```

## When to Use What

| Cache | Best For | Pros | Cons |
|-------|----------|------|------|
| **Redis** | Object cache, sessions, full-page cache | Persistence, data types, replication | Slightly more RAM |
| **Memcached** | Simple key-value cache | Simpler, lower memory overhead | No persistence, no replication |

## Flush Cache

```bash
# Redis
redis-cli FLUSHALL

# Memcached
echo "flush_all" | nc -q1 localhost 11211

# WordPress (redis-cache plugin)
wp cache flush

# Redis CLI specific DB
redis-cli -n 0 FLUSHDB
```

## Performance Test

```bash
# Redis benchmark
redis-benchmark -q -n 1000

# Memcached benchmark
memcached-tool localhost:11211 stats
```

## Monitoring

```bash
# Redis
redis-cli info stats
redis-cli info memory
redis-cli info keyspace

# Memcached
echo "stats" | nc -q1 localhost 11211
```

## Troubleshooting

| Symptom | Cause | Fix |
|---------|-------|-----|
| Cache not working | PHP extension missing | `apt install php-redis` |
| Connection refused | Redis not running | `systemctl status redis` |
| Out of memory | `maxmemory` too low | Increase in redis.conf |
| Stale cache | TTL too long | Clear cache, reduce TTL |
