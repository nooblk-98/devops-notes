# Node.js / PM2 Deployment

## Prerequisites

```bash
# Install Node.js
curl -fsSL https://deb.nodesource.com/setup_18.x | bash -
apt install -y nodejs

# Install PM2 globally
npm install -g pm2
```

## PM2 Basics

```bash
# Start app
pm2 start app.js --name my-app

# Start with ecosystem file
pm2 start ecosystem.config.js

# List all processes
pm2 list

# Stop
pm2 stop my-app

# Restart
pm2 restart my-app

# Delete
pm2 delete my-app

# View logs
pm2 logs my-app
pm2 logs --lines=100

# Monitor
pm2 monit
pm2 status
```

## Ecosystem File

```js title="ecosystem.config.js"
module.exports = {
  apps: [{
    name: 'api',
    script: 'dist/server.js',
    instances: 'max',           // Cluster mode (all CPUs)
    exec_mode: 'cluster',
    env: {
      NODE_ENV: 'development',
      PORT: 3001
    },
    env_production: {
      NODE_ENV: 'production',
      PORT: 3000
    },
    watch: false,
    max_memory_restart: '512M',
    log_date_format: 'YYYY-MM-DD HH:mm:ss',
    error_file: './logs/err.log',
    out_file: './logs/out.log',
    merge_logs: true,
    autorestart: true,
    max_restarts: 10,
    restart_delay: 4000
  }]
}
```

```bash
# Start with production env
pm2 start ecosystem.config.js --env production

# Reload (zero-downtime)
pm2 reload ecosystem.config.js --env production
```

## Nginx Reverse Proxy

```nginx
server {
    listen 443 ssl http2;
    server_name api.example.com;

    location / {
        proxy_pass http://127.0.0.1:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_cache_bypass $http_upgrade;
    }
}
```

## Startup on Boot

```bash
# Generate systemd service
pm2 startup systemd -u www-data

# Save current process list
pm2 save

# Verify
systemctl status pm2-www-data
```

## Docker Compose with PM2

```yaml
services:
  app:
    image: node:18-alpine
    working_dir: /app
    command: npx pm2-runtime start ecosystem.config.js --env production
    ports:
      - "127.0.0.1:3000:3000"
    volumes:
      - ./app:/app
    environment:
      - NODE_ENV=production
```

## Multi-App Setup

```js title="ecosystem.config.js"
module.exports = {
  apps: [
    {
      name: 'api',
      script: 'api/dist/server.js',
      instances: 2,
      env_production: { PORT: 3000 }
    },
    {
      name: 'worker',
      script: 'worker/index.js',
      instances: 1,
      env_production: { PORT: 3001 }
    },
    {
      name: 'websocket',
      script: 'ws/server.js',
      instances: 1,
      env_production: { PORT: 3002 }
    }
  ]
}
```

## Deploy with PM2

```bash title="ecosystem.config.js"
module.exports = {
  apps: [{ name: 'app', script: 'server.js' }],
  deploy: {
    production: {
      user: 'deploy',
      host: 'server.example.com',
      ref: 'origin/main',
      repo: 'git@github.com:org/app.git',
      path: '/opt/app',
      'post-deploy': 'npm ci && npm run build && pm2 reload ecosystem.config.js --env production'
    }
  }
}
```

```bash
pm2 deploy production setup
pm2 deploy production
```

## GitHub Actions (PM2 Deploy)

```yaml
name: Deploy Node.js

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Deploy via SSH
        uses: appleboy/ssh-action@v1
        with:
          host: ${{ secrets.HOST }}
          username: ${{ secrets.USER }}
          key: ${{ secrets.SSH_KEY }}
          script: |
            cd /opt/app
            git pull origin main
            npm ci --production
            npm run build
            pm2 reload ecosystem.config.js --env production
```

## Monitoring

```bash
# Process list
pm2 ls

# Real-time metrics
pm2 monit

# CPU/Memory per process
pm2 prettylist

# API metrics
pm2 web 9615

# Logs
pm2 logs --lines=50
```

## Troubleshooting

| Symptom | Cause | Fix |
|---------|-------|-----|
| Port in use | Another process on same port | `kill $(lsof -ti:3000)` |
| Out of memory | Memory leak | Set `max_memory_restart` |
| "EADDRINUSE" | Port conflict | Change PORT in ecosystem |
| App not restarting | Max restarts exceeded | `pm2 reset app && pm2 restart app` |
| Zero-downtime fails | Health check missing | Add `-i` flag or `listen_timeout` |

## Verification

- [ ] App starts with `pm2 start`
- [ ] Cluster mode enabled (multi-core)
- [ ] Startup on boot configured
- [ ] Log rotation configured
- [ ] Max memory restart set
- [ ] Nginx reverse proxy working
- [ ] Deploy via GitHub Actions works
