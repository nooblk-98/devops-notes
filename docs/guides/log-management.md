# Log Management

## Logrotate Configuration

### Nginx Logs

```nginx title="/etc/logrotate.d/nginx"
/var/log/nginx/*.log {
    daily
    missingok
    rotate 30
    compress
    delaycompress
    notifempty
    sharedscripts
    postrotate
        systemctl reload nginx > /dev/null 2>&1 || true
    endscript
}
```

### MySQL Logs

```nginx title="/etc/logrotate.d/mysql"
/var/log/mysql/*.log {
    daily
    rotate 30
    compress
    missingok
    notifempty
    sharedscripts
    postrotate
        systemctl reload mysql > /dev/null 2>&1 || true
    endscript
}
```

### System Logs

```nginx title="/etc/logrotate.d/rsyslog"
/var/log/syslog
/var/log/auth.log
/var/log/kern.log {
    weekly
    rotate 12
    compress
    delaycompress
    missingok
    notifempty
}
```

## Centralized Logging with Loki + Promtail

### Install Promtail

```bash
# Download Promtail
wget https://github.com/grafana/loki/releases/latest/download/promtail-linux-amd64.zip
unzip promtail-linux-amd64.zip
sudo mv promtail-linux-amd64 /usr/local/bin/promtail

# Create config
mkdir -p /etc/promtail
```

### Promtail Config

```yaml title="/etc/promtail/promtail.yml"
server:
  http_listen_port: 9080
  grpc_listen_port: 0

positions:
  filename: /tmp/positions.yaml

clients:
  - url: http://loki.example.com:3100/loki/api/v1/push

scrape_configs:
  - job_name: nginx
    static_configs:
      - targets: [localhost]
        labels:
          job: nginx
          __path__: /var/log/nginx/*.log

  - job_name: syslog
    static_configs:
      - targets: [localhost]
        labels:
          job: syslog
          __path__: /var/log/syslog
```

### Promtail Systemd Service

```ini title="/etc/systemd/system/promtail.service"
[Unit]
Description=Promtail service
After=network.target

[Service]
Type=simple
User=root
ExecStart=/usr/local/bin/promtail -config.file=/etc/promtail/promtail.yml
Restart=always

[Install]
WantedBy=multi-user.target
```

```bash
systemctl daemon-reload
systemctl enable --now promtail
```

## Centralized Logging with ELK (Filebeat + Elasticsearch + Kibana)

### Filebeat Install

```bash
curl -fsSL https://artifacts.elastic.co/GPG-KEY-elasticsearch | apt-key add -
echo "deb https://artifacts.elastic.co/packages/8.x/apt stable main" | tee /etc/apt/sources.list.d/elastic-8.x.list
apt update && apt install -y filebeat
```

### Filebeat Config

```yaml title="/etc/filebeat/filebeat.yml"
filebeat.inputs:
  - type: log
    enabled: true
    paths:
      - /var/log/nginx/access.log
      - /var/log/nginx/error.log
    fields:
      service: nginx

  - type: log
    enabled: true
    paths:
      - /var/log/mysql/*.log
    fields:
      service: mysql

output.elasticsearch:
  hosts: ["http://elasticsearch.example.com:9200"]
  username: "filebeat"
  password: "password"

setup.kibana:
  host: "http://kibana.example.com:5601"
```

## Docker Container Logs

```bash
# View logs
docker logs -f --tail 100 <container>

# Limit log size (in docker-compose.yml)
service:
  logging:
    driver: json-file
    options:
      max-size: "10m"
      max-file: "3"

# Clean all container logs
truncate -s 0 /var/lib/docker/containers/*/*-json.log

# Global Docker log config
# /etc/docker/daemon.json
{
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "10m",
    "max-file": "3"
  }
}
```

## Checking Logs

```bash
# Real-time monitoring
tail -f /var/log/nginx/access.log
journalctl -u nginx -f

# Search logs
grep "ERROR" /var/log/nginx/error.log
journalctl -u mysql --since "1 hour ago" | grep -i error

# View last boot logs
journalctl -b

# Disk usage by logs
du -sh /var/log/*
```
