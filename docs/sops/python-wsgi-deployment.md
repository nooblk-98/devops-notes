# Python / WSGI Deployment SOP

## Architecture

```
User → Nginx (host) → Gunicorn container (8000) → PostgreSQL container (5432)
```

## Project Structure

```
project/
├── app/
├── config/
├── manage.py / wsgi.py
├── requirements.txt
├── Dockerfile
├── docker-compose.yml
└── .github/workflows/deploy.yml
```

## Dockerfile (Django)

```dockerfile title="Dockerfile"
FROM python:3.11-slim AS base

WORKDIR /app

RUN apt update && apt install -y --no-install-recommends \
    gcc libpq-dev && rm -rf /var/lib/apt/lists/*

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

RUN python manage.py collectstatic --noinput

EXPOSE 8000

CMD ["gunicorn", "config.wsgi:application", "--bind", "0.0.0.0:8000", "--workers", "4"]
```

## Dockerfile (Flask)

```dockerfile title="Dockerfile"
FROM python:3.11-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

EXPOSE 8000

CMD ["gunicorn", "app:app", "--bind", "0.0.0.0:8000", "--workers", "4"]
```

## Docker Compose

```yaml title="docker-compose.yml"
services:
  db:
    image: postgres:15
    container_name: python-db
    restart: unless-stopped
    env_file: .env
    volumes:
      - ./db:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U $${POSTGRES_USER} -d $${POSTGRES_DB}"]
      interval: 10s
      timeout: 5s
      retries: 5

  app:
    build: .
    container_name: python-app
    restart: unless-stopped
    ports:
      - "127.0.0.1:8000:8000"
    volumes:
      - static:/app/static
      - media:/app/media
    env_file: .env
    depends_on:
      db:
        condition: service_healthy
    command: >
      sh -c "python manage.py migrate --noinput &&
             gunicorn config.wsgi:application --bind 0.0.0.0:8000 --workers 4"

  redis:
    image: redis:alpine
    container_name: python-redis
    restart: unless-stopped

volumes:
  static:
  media:
```

## Environment File

```bash title=".env"
DEBUG=False
SECRET_KEY=your-secret-key
ALLOWED_HOSTS=example.com,api.example.com

DB_ENGINE=django.db.backends.postgresql
DB_NAME=myapp
DB_USER=myapp_user
DB_PASSWORD=strong_password
DB_HOST=db
DB_PORT=5432

POSTGRES_DB=myapp
POSTGRES_USER=myapp_user
POSTGRES_PASSWORD=strong_password

REDIS_URL=redis://redis:6379/0

EMAIL_HOST=smtp.example.com
EMAIL_PORT=587
EMAIL_HOST_USER=admin@example.com
EMAIL_HOST_PASSWORD=password
```

## Nginx Configuration

```nginx
server {
    listen 443 ssl http2;
    server_name api.example.com;

    client_max_body_size 100M;

    location /static/ {
        alias /opt/python-app/static/;
        expires 30d;
    }

    location /media/ {
        alias /opt/python-app/media/;
        expires 7d;
    }

    location / {
        proxy_pass http://127.0.0.1:8000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

## Django-Specific

### Static Files

```bash
# Collect static files
docker compose exec app python manage.py collectstatic --noinput

# Check
docker compose exec app ls static/
```

### Migrations

```bash
# Auto-apply on startup (set in docker-compose command)
# Or run manually:
docker compose exec app python manage.py migrate --noinput

# Create migrations (dev only)
docker compose exec app python manage.py makemigrations
```

### Management Commands

```bash
# Django shell
docker compose exec app python manage.py shell

# Create superuser
docker compose exec app python manage.py createsuperuser

# Dump data
docker compose exec app python manage.py dumpdata > data.json

# Load data
docker compose exec app python manage.py loaddata data.json
```

## Gunicorn Tuning

```bash
# Workers = (2 × CPU cores) + 1
gunicorn config.wsgi:application \
  --bind 0.0.0.0:8000 \
  --workers 4 \
  --worker-class sync \
  --timeout 120 \
  --max-requests 1000 \
  --max-requests-jitter 50 \
  --log-level info \
  --access-logfile - \
  --error-logfile -
```

### Systemd Service (Without Docker)

```ini title="/etc/systemd/system/gunicorn.service"
[Unit]
Description=Gunicorn daemon
After=network.target

[Service]
User=www-data
Group=www-data
WorkingDirectory=/var/www/app
ExecStart=/usr/local/bin/gunicorn config.wsgi:application \
  --workers 4 \
  --bind unix:/run/gunicorn.sock

[Install]
WantedBy=multi-user.target
```

## GitHub Actions

```yaml
name: Deploy Django

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
            cd /opt/python-app
            git pull origin main
            docker compose build
            docker compose up -d --force-recreate
            docker compose exec -T app python manage.py migrate --noinput
            docker compose exec -T app python manage.py collectstatic --noinput
            docker image prune -f
```

## Verification

- [ ] Docker containers running
- [ ] Gunicorn listening on port 8000
- [ ] Nginx proxies to Gunicorn
- [ ] Static files served correctly
- [ ] Database migrations applied
- [ ] Admin accessible
- [ ] Media uploads working
- [ ] GitHub Actions deploy succeeds
