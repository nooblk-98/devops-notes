# Monitoring Agent Setup

## Node Exporter (Server Metrics)

### Install

```bash
# Download
wget https://github.com/prometheus/node_exporter/releases/latest/download/node_exporter-linux-amd64.tar.gz
tar xzf node_exporter-linux-amd64.tar.gz
sudo mv node_exporter-linux-amd64/node_exporter /usr/local/bin/
rm -rf node_exporter-linux-amd64*
```

### Systemd Service

```ini title="/etc/systemd/system/node_exporter.service"
[Unit]
Description=Node Exporter
After=network.target

[Service]
Type=simple
User=nobody
Group=nogroup
ExecStart=/usr/local/bin/node_exporter \
  --web.listen-address=:9100 \
  --collector.systemd \
  --collector.textfile.directory=/var/lib/node_exporter

Restart=always

[Install]
WantedBy=multi-user.target
```

```bash
systemctl daemon-reload
systemctl enable --now node_exporter
```

### Verify

```bash
curl http://localhost:9100/metrics | head -20
```

## MySQL Exporter

```bash
# Download
wget https://github.com/prometheus/mysqld_exporter/releases/latest/download/mysqld_exporter-linux-amd64.tar.gz
tar xzf mysqld_exporter-linux-amd64.tar.gz
sudo mv mysqld_exporter-linux-amd64/mysqld_exporter /usr/local/bin/
```

```ini title="/etc/systemd/system/mysqld_exporter.service"
[Unit]
Description=MySQL Exporter
After=network.target mysql.service

[Service]
Type=simple
User=mysql
ExecStart=/usr/local/bin/mysqld_exporter \
  --web.listen-address=:9104 \
  --config.my-cnf=/etc/.mysqld_exporter.cnf

Restart=always

[Install]
WantedBy=multi-user.target
```

```bash
systemctl daemon-reload
systemctl enable --now mysqld_exporter
```

## Nginx Exporter

```bash
# Download
wget https://github.com/nginxinc/nginx-prometheus-exporter/releases/latest/download/nginx-prometheus-exporter_linux-amd64.tar.gz
tar xzf nginx-prometheus-exporter_linux-amd64.tar.gz
sudo mv nginx-prometheus-exporter /usr/local/bin/
```

```ini title="/etc/systemd/system/nginx_exporter.service"
[Unit]
Description=Nginx Exporter
After=network.target nginx.service

[Service]
Type=simple
ExecStart=/usr/local/bin/nginx-prometheus-exporter \
  --web.listen-address=:9113 \
  --nginx.scrape-uri=http://localhost/nginx_status

Restart=always

[Install]
WantedBy=multi-user.target
```

## Prometheus Server

### Install

```bash
wget https://github.com/prometheus/prometheus/releases/latest/download/prometheus-linux-amd64.tar.gz
tar xzf prometheus-linux-amd64.tar.gz
sudo mv prometheus-linux-amd64/{prometheus,promtool} /usr/local/bin/
sudo mkdir -p /etc/prometheus /var/lib/prometheus
sudo mv prometheus-linux-amd64/{consoles,console_libraries} /etc/prometheus/
```

### Config

```yaml title="/etc/prometheus/prometheus.yml"
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: node
    static_configs:
      - targets:
        - server1.example.com:9100
        - server2.example.com:9100

  - job_name: mysql
    static_configs:
      - targets:
        - db1.example.com:9104

  - job_name: nginx
    static_configs:
      - targets:
        - web1.example.com:9113
```

### Systemd Service

```ini title="/etc/systemd/system/prometheus.service"
[Unit]
Description=Prometheus
After=network.target

[Service]
Type=simple
User=prometheus
Group=prometheus
ExecStart=/usr/local/bin/prometheus \
  --config.file=/etc/prometheus/prometheus.yml \
  --storage.tsdb.path=/var/lib/prometheus

Restart=always

[Install]
WantedBy=multi-user.target
```

## Grafana

### Install

```bash
apt-get install -y software-properties-common wget
wget -q -O - https://packages.grafana.com/gpg.key | apt-key add -
add-apt-repository "deb https://packages.grafana.com/oss/deb stable main"
apt update && apt install -y grafana

systemctl enable --now grafana-server
```

### Default Data Source

```yaml title="/etc/grafana/provisioning/datasources/prometheus.yml"
apiVersion: 1
datasources:
  - name: Prometheus
    type: prometheus
    access: proxy
    url: http://localhost:9090
    isDefault: true
```

Access Grafana at `http://<server-ip>:3000` (default: admin/admin).

## Alertmanager

### Install

```bash
wget https://github.com/prometheus/alertmanager/releases/latest/download/alertmanager-linux-amd64.tar.gz
tar xzf alertmanager-linux-amd64.tar.gz
sudo mv alertmanager-linux-amd64/{alertmanager,amtool} /usr/local/bin/
```

### Config

```yaml title="/etc/alertmanager/alertmanager.yml"
route:
  receiver: email-alerts
  group_by: [alertname]
  group_wait: 30s
  group_interval: 5m
  repeat_interval: 4h

receivers:
  - name: email-alerts
    email_configs:
      - to: ops@example.com
        from: alertmanager@example.com
        smarthost: smtp.example.com:587
        auth_username: user
        auth_password: password
```

## Verification

- [ ] Node Exporter metrics accessible on `:9100/metrics`
- [ ] MySQL Exporter metrics on `:9104/metrics`
- [ ] Prometheus targets all UP
- [ ] Grafana dashboards show data
- [ ] Alerts fire correctly
