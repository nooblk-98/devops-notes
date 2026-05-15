# Nginx Performance Tuning

## Worker Processes

```nginx title="/etc/nginx/nginx.conf"
worker_processes auto;           # One per CPU core
worker_rlimit_nofile 65535;       # File descriptor limit
```

```bash
# Check CPU cores
nproc

# Verify worker count
ps aux | grep nginx | grep "worker process" | wc -l
```

## Worker Connections

```nginx
events {
    worker_connections 4096;       # Max connections per worker
    multi_accept on;               # Accept all connections at once
    use epoll;                     # Linux efficient event model
}
```

### Max Connections Formula

```
max_clients = worker_processes × worker_connections
```

Example: `4 × 4096 = 16,384` concurrent clients.

## Buffers

```nginx
http {
    # Client body buffer
    client_body_buffer_size 128k;
    client_max_body_size 64m;

    # Header buffer
    client_header_buffer_size 1k;
    large_client_header_buffers 4 8k;

    # Proxy / FastCGI buffers
    proxy_buffers 16 16k;
    proxy_buffer_size 32k;
    proxy_busy_buffers_size 256k;

    fastcgi_buffers 16 16k;
    fastcgi_buffer_size 32k;
    fastcgi_busy_buffers_size 256k;

    # Output buffers
    output_buffers 32 32k;
    postpone_output 1460;
}
```

## Timeouts

```nginx
http {
    keepalive_timeout 30;
    keepalive_requests 100;
    client_body_timeout 30;
    client_header_timeout 30;
    send_timeout 10;

    # Proxy / FastCGI timeouts
    proxy_connect_timeout 30;
    proxy_send_timeout 60;
    proxy_read_timeout 60;

    fastcgi_connect_timeout 30;
    fastcgi_send_timeout 60;
    fastcgi_read_timeout 60;
}
```

## Gzip Compression

```nginx
http {
    gzip on;
    gzip_vary on;
    gzip_proxied any;
    gzip_comp_level 5;
    gzip_min_length 256;
    gzip_types
        text/plain
        text/css
        text/javascript
        application/javascript
        application/json
        application/xml
        image/svg+xml
        font/woff
        font/woff2;
}
```

## Static File Caching

```nginx
server {
    # Browser cache for static files
    location ~* \.(jpg|jpeg|png|gif|ico|webp|svg|css|js|woff|woff2|ttf|eot)$ {
        expires 30d;
        add_header Cache-Control "public, immutable";
        access_log off;
        log_not_found off;
    }

    # No cache for HTML
    location / {
        expires -1;
        add_header Cache-Control "no-store, no-cache, must-revalidate";
    }
}
```

## SSL Performance

```nginx
server {
    listen 443 ssl http2;
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384;
    ssl_prefer_server_ciphers on;
    ssl_session_cache shared:SSL:10m;
    ssl_session_timeout 10m;
    ssl_session_tickets off;
    ssl_buffer_size 4k;
}
```

## Logging

```nginx
http {
    # Reduce I/O — only log errors in production
    access_log off;
    # Or buffer logs
    access_log /var/log/nginx/access.log buffer=32k flush=5s;

    # Error log level
    error_log /var/log/nginx/error.log warn;
}
```

## Rate Limiting

```nginx
http {
    # Define zone
    limit_req_zone $binary_remote_addr zone=login:10m rate=3r/m;
    limit_req_zone $binary_remote_addr zone=api:10m rate=30r/m;

    server {
        location /wp-login.php {
            limit_req zone=login burst=5 nodelay;
            # ... rest of config
        }

        location /api/ {
            limit_req zone=api burst=10 nodelay;
            # ... rest of config
        }
    }
}
```

## Full Recommended Config

```nginx title="/etc/nginx/nginx.conf"
user www-data;
worker_processes auto;
worker_rlimit_nofile 65535;
pid /run/nginx.pid;

events {
    worker_connections 4096;
    multi_accept on;
    use epoll;
}

http {
    sendfile on;
    tcp_nopush on;
    tcp_nodelay on;
    keepalive_timeout 30;
    keepalive_requests 100;
    types_hash_max_size 2048;
    server_tokens off;

    include /etc/nginx/mime.types;
    default_type application/octet-stream;

    # Buffers
    client_body_buffer_size 128k;
    client_max_body_size 64m;
    client_header_buffer_size 1k;
    large_client_header_buffers 4 8k;

    # Timeouts
    client_body_timeout 30;
    client_header_timeout 30;
    send_timeout 10;

    # Gzip
    gzip on;
    gzip_vary on;
    gzip_comp_level 5;
    gzip_min_length 256;
    gzip_types text/plain text/css text/javascript application/javascript application/json application/xml image/svg+xml;

    # Logging
    access_log /var/log/nginx/access.log buffer=32k flush=5s;
    error_log /var/log/nginx/error.log warn;

    # Rate limit zones
    limit_req_zone $binary_remote_addr zone=login:10m rate=3r/m;

    include /etc/nginx/conf.d/*.conf;
    include /etc/nginx/sites-enabled/*;
}
```

## Apply & Test

```bash
nginx -t && systemctl reload nginx
```

## Benchmark

```bash
# Install and run
apt install -y apache2-utils
ab -n 1000 -c 50 https://example.com/

# Check results
# Requests per second
# Time per request
# Transfer rate
```

## Verification

- [ ] `worker_processes auto` set
- [ ] `worker_connections` adequate for expected traffic
- [ ] Gzip enabled
- [ ] Static files cached (30d)
- [ ] SSL session cache configured
- [ ] Rate limiting on login/admin
- [ ] Server tokens off
- [ ] Access log buffered
