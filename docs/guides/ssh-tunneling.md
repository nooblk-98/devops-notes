# SSH Tunneling

## Local Port Forwarding

Forward a remote port to your local machine:

```bash
# Remote MySQL (3306) → localhost:3307
ssh -L 3307:localhost:3306 user@server

# Remote service on specific host
ssh -L 8080:internal-server:80 user@bastion

# Multiple tunnels
ssh -L 3307:localhost:3306 -L 6379:localhost:6379 user@server
```

**Use case:** Access a database that only listens on localhost.

## Remote Port Forwarding

Forward your local port to the remote server:

```bash
# Local web server → remote:8080
ssh -R 8080:localhost:3000 user@server

# Multiple
ssh -R 8080:localhost:3000 -R 9090:localhost:9000 user@server
```

**Use case:** Share your local dev server with a colleague via a public server.

## Dynamic Port Forwarding (SOCKS Proxy)

```bash
# Create SOCKS proxy on localhost:1080
ssh -D 1080 user@server

# Then configure browser/application to use SOCKS5 proxy:
# Host: localhost
# Port: 1080
```

**Use case:** Browse the web through the remote server's network (bypass geo-restrictions).

## Jump Host (Bastion)

```bash
# Connect to internal server via bastion
ssh -J user@bastion user@internal-server

# In SSH config (~/.ssh/config)
Host internal
    HostName 10.0.1.50
    User admin
    ProxyJump user@bastion.example.com
    ForwardAgent yes

# Then: ssh internal
```

**Use case:** Reach servers that are not directly accessible from the internet.

## SSH Config for Tunnels

```ini title="~/.ssh/config"
# Database tunnel
Host db-tunnel
    HostName db-server.example.com
    User devops
    LocalForward 3307 localhost:3306
    LocalForward 6379 localhost:6379

# Persistent tunnel (auto-reconnect)
Host persistent-tunnel
    HostName server.example.com
    User devops
    ServerAliveInterval 60
    ServerAliveCountMax 3
    ExitOnForwardFailure yes
    LocalForward 3307 localhost:3306
    LocalForward 8080 localhost:80
```

## Auto-Reconnect Tunnel

```bash title="/usr/local/bin/ssh-tunnel.sh"
#!/bin/bash
while true; do
    ssh -o ServerAliveInterval=30 \
        -o ServerAliveCountMax=3 \
        -o ExitOnForwardFailure=yes \
        -L 3307:localhost:3306 \
        -L 6379:localhost:6379 \
        user@server
    sleep 5
done
```

### Systemd Service

```ini title="/etc/systemd/system/ssh-tunnel.service"
[Unit]
Description=SSH Tunnel
After=network.target

[Service]
Type=simple
User=devops
ExecStart=/usr/local/bin/ssh-tunnel.sh
Restart=always
RestartSec=10

[Install]
WantedBy=multi-user.target
```

```bash
systemctl daemon-reload
systemctl enable --now ssh-tunnel
```

## Reverse Tunnel (Expose Local to Internet)

```bash
# On local machine: forward local:3000 → server:8080
ssh -R 8080:localhost:3000 user@public-server

# Now anyone accessing public-server:8080 sees your local app
```

### Auto-SSH for Reverse Tunnel

```bash
autossh -M 0 -o "ServerAliveInterval 30" -o "ServerAliveCountMax 3" \
  -R 8080:localhost:3000 user@public-server -N
```

## Tunnel Through HTTPS Proxy

```bash
# Using corkscrew or netcat
ssh -o "ProxyCommand corkscrew proxy.example.com 3128 %h %p" user@server

# With netcat
ssh -o "ProxyCommand nc -X connect -x proxy.example.com:3128 %h %p" user@server
```

## Copy Files Through Tunnel

```bash
# SCP through jump host
scp -o "ProxyJump user@bastion" file.txt user@internal-server:/tmp/

# Rsync through tunnel
rsync -avz -e "ssh -J user@bastion" ./ user@internal-server:/var/www/
```

## Knowledge Base Access

```bash
# Access web UI on remote server
ssh -L 8080:localhost:8080 user@server
# Then open http://localhost:8080

# Access internal service through bastion
ssh -J bastion -L 9090:internal-web:80 user@bastion
# Then open http://localhost:9090
```

## Common Use Cases

| Tunnel | Command | Purpose |
|--------|---------|---------|
| MySQL | `-L 3306:localhost:3306` | Access remote MySQL locally |
| Redis | `-L 6379:localhost:6379` | Debug Redis from local |
| Adminer | `-L 8080:localhost:80` | Access internal web UI |
| Kubernetes | `-L 6443:localhost:6443` | Control remote cluster |
| Prometheus | `-L 9090:localhost:9090` | View remote metrics |
| Grafana | `-L 3000:localhost:3000` | View remote dashboards |

## Troubleshooting

```bash
# Verbose tunnel debug
ssh -vvv -L 3307:localhost:3306 user@server

# Check if tunnel port is listening
ss -tlnp | grep 3307

# Kill hanging tunnel
lsof -ti:3307 | xargs kill

# GatewayPorts (allow others to use your tunnel)
ssh -R 0.0.0.0:8080:localhost:3000 user@server
# Requires GatewayPorts yes in sshd_config
```

## Security

```ini title="/etc/ssh/sshd_config"
# Allow only specific users to create tunnels
AllowTcpForwarding yes
GatewayPorts no
PermitOpen localhost:3306 localhost:6379
```

## Verification

- [ ] Tunnel established (can connect to forwarded port)
- [ ] Auto-reconnect configured (autossh/systemd)
- [ ] Jump host working for internal servers
- [ ] Tunnels restart on boot
- [ ] Only necessary ports forwarded
