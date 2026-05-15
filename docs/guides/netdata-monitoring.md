# Monitoring with Netdata

## Installation

### One-Line Install

```bash
bash <(curl -Ss https://my-netdata.io/kickstart.sh)
```

### Docker Install

```yaml title="docker-compose.yml"
services:
  netdata:
    image: netdata/netdata:latest
    container_name: netdata
    hostname: server-name
    restart: unless-stopped
    ports:
      - "127.0.0.1:19999:19999"
    cap_add:
      - SYS_PTRACE
    security_opt:
      - apparmor:unconfined
    volumes:
      - netdataconfig:/etc/netdata
      - netdatalib:/var/lib/netdata
      - netdatacache:/var/cache/netdata
      - /etc/passwd:/host/etc/passwd:ro
      - /etc/group:/host/etc/group:ro
      - /proc:/host/proc:ro
      - /sys:/host/sys:ro
      - /etc/os-release:/host/etc/os-release:ro

volumes:
  netdataconfig:
  netdatalib:
  netdatacache:
```

## Access Dashboard

```
http://<server-ip>:19999
```

## Nginx Reverse Proxy

```nginx
server {
    listen 443 ssl http2;
    server_name netdata.example.com;

    location / {
        proxy_pass http://127.0.0.1:19999;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

## Basic Auth

```bash
# Install password tool
apt install -y apache2-utils

# Create user
htpasswd -c /etc/netdata/netdata-access admin

# Restart Netdata
systemctl restart netdata
```

## Enable Plugins

```bash
# Edit netdata config
/etc/netdata/netdata.conf

# Enable specific plugins
[plugin:proc]
    /proc/loadavg = yes
    /proc/stat = yes
    /proc/meminfo = yes
    /proc/net/dev = yes
    /proc/diskstats = yes
```

## Alarms & Notifications

### Configure Notifications

Edit `/etc/netdata/health_alarm_notify.conf`:

```bash
# Email
SEND_EMAIL="YES"
DEFAULT_RECIPIENT_EMAIL="admin@example.com"

# Slack
SEND_SLACK="YES"
SLACK_WEBHOOK_URL="https://hooks.slack.com/services/xxx/yyy/zzz"
DEFAULT_RECIPIENT_SLACK="#alerts"

# Discord
SEND_DISCORD="YES"
DISCORD_WEBHOOK_URL="https://discord.com/api/webhooks/xxx/yyy"
DEFAULT_RECIPIENT_DISCORD="#alerts"
```

### Custom Alarms

Create `/etc/netdata/health.d/custom.conf`:

```bash
# CPU > 80% for 5 minutes
template: cpu_usage
    on: system.cpu
    lookup: average -5m percentage
    units: %
    every: 30s
    warn: $this > 80
    crit: $this > 95
    info: CPU usage
    to: sysadmin

# Disk space > 85%
template: disk_space
    on: disk.space
    lookup: used -10s absolute
    units: GB
    every: 30s
    warn: $this > 85
    crit: $this > 95
    info: Disk space usage
```

## Monitored Metrics

| Category | Metrics |
|----------|---------|
| CPU | Usage per core, frequency, temperature |
| Memory | RAM, swap, available, cached |
| Disk | Usage, IOPS, latency, throughput |
| Network | Bandwidth, packets, errors, drops |
| Processes | Running, blocked, zombies |
| Services | Nginx, MySQL, Redis, PostgreSQL |
| Containers | Docker, individual containers |

## Command Line

```bash
# Show real-time metrics
netdata -W set "httpd" status monitoring

# Health checks
netdatacli check

# View alarms
netdatacli show-alarms all
```

## Backup Config

```bash
tar czf netdata-config-backup.tar.gz /etc/netdata/
```

## Update

```bash
# Native install
bash <(curl -Ss https://my-netdata.io/kickstart.sh) --no-updates

# Docker
docker compose pull netdata
docker compose up -d netdata
```

## Verification

- [ ] Dashboard accessible at `http://<ip>:19999`
- [ ] CPU, memory, disk, network metrics visible
- [ ] Alarms configured for critical metrics
- [ ] Notifications working (Slack/Email)
- [ ] Nginx proxy with basic auth
- [ ] Netdata auto-starts on boot
