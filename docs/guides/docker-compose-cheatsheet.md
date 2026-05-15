# Docker Compose Cheat Sheet

## Service Lifecycle

```bash
# Start services
docker compose up -d

# Build and start
docker compose up -d --build

# Recreate containers (force)
docker compose up -d --force-recreate

# Stop services
docker compose down

# Stop and remove volumes
docker compose down -v

# Restart service
docker compose restart <service>

# View logs
docker compose logs -f --tail=100

# Follow logs for specific service
docker compose logs -f <service>

# List services
docker compose ps

# Execute command in service
docker compose exec <service> <command>

# Run one-off command
docker compose run --rm <service> <command>
```

## Images

```bash
# Pull latest images
docker compose pull

# Build services
docker compose build

# Build without cache
docker compose build --no-cache

# Build specific service
docker compose build <service>
```

## Healthchecks

```yaml
services:
  app:
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost/health"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 15s
```

### Common Healthchecks

| Service | Healthcheck |
|---------|-------------|
| MariaDB/MySQL | `healthcheck.sh --connect --innodb_initialized` |
| PostgreSQL | `pg_isready -U postgres` |
| Redis | `redis-cli ping` |
| Nginx | `curl -f http://localhost/health` |
| Node.js | `curl -f http://localhost:3000/health` |

## Resource Limits

```yaml
services:
  app:
    deploy:
      resources:
        limits:
          cpus: "0.5"
          memory: "256M"
        reservations:
          cpus: "0.25"
          memory: "128M"
```

## Networking

### Custom Network

```yaml
services:
  app:
    networks:
      - frontend
      - backend
  db:
    networks:
      - backend

networks:
  frontend:
    driver: bridge
  backend:
    driver: bridge
```

### Hostname & Aliases

```yaml
services:
  app:
    hostname: app
    networks:
      backend:
        aliases:
          - api
          - app-internal
```

### Static IP

```yaml
services:
  app:
    networks:
      backend:
        ipv4_address: 172.20.0.10

networks:
  backend:
    driver: bridge
    ipam:
      config:
        - subnet: 172.20.0.0/16
```

## Volumes

### Named vs Bind Mounts

```yaml
services:
  app:
    volumes:
      - app-data:/var/www/html        # Named volume (managed by Docker)
      - ./html:/var/www/html          # Bind mount (host path)
      - ./config.yml:/app/config.yml  # File bind mount

volumes:
  app-data:
```

### Volume Labels & Driver

```yaml
volumes:
  app-data:
    driver: local
    labels:
      backup: "true"
      environment: "production"
```

## Environment Variables

```yaml
services:
  app:
    environment:
      - NODE_ENV=production
      - DB_HOST=db
    env_file:
      - .env
      - .env.production
```

## Dependencies

```yaml
services:
  app:
    depends_on:
      - db
    # Conditional:
    depends_on:
      db:
        condition: service_healthy
      redis:
        condition: service_started
```

## Logging

```yaml
services:
  app:
    logging:
      driver: json-file
      options:
        max-size: "10m"
        max-file: "3"
```

## Restart Policies

```yaml
services:
  app:
    restart: unless-stopped
    # Options: no, always, on-failure:5, unless-stopped
```

## Profiles (Conditional Services)

```yaml
services:
  redis:
    image: redis:alpine
    profiles:
      - cache
      - dev

  mailhog:
    image: mailhog/mailhog
    profiles:
      - dev
```

```bash
docker compose --profile dev up -d
```

## Multi-File Setup

```
├── docker-compose.yml        # Base config
├── docker-compose.override.yml # Dev overrides (auto-loaded)
├── docker-compose.prod.yml   # Production overrides
```

```bash
docker compose -f docker-compose.yml -f docker-compose.prod.yml up -d
```

## Secrets

```yaml
services:
  app:
    secrets:
      - db_password

secrets:
  db_password:
    file: ./secrets/db_password.txt
```

## Common Patterns

### WordPress

```yaml
services:
  wordpress:
    image: wordpress:php8.2-fpm
    volumes: ["./html:/var/www/html"]
  db:
    image: mariadb:10
    volumes: ["./db:/var/lib/mysql"]
```

### Laravel

```yaml
services:
  app:
    build: .
    ports: ["127.0.0.1:9000:9000"]
    volumes: ["./html:/var/www/html"]
  db:
    image: mariadb:10
    volumes: ["./db:/var/lib/mysql"]
  redis:
    image: redis:alpine
    volumes: ["./redis:/data"]
```

### Node.js

```yaml
services:
  app:
    build: .
    ports: ["127.0.0.1:3000:3000"]
    environment: [NODE_ENV=production]
```

## Useful One-Liners

```bash
# Zero-downtime: recreate single service
docker compose up -d --no-deps --force-recreate <service>

# Scale service
docker compose up -d --scale app=3

# Check config
docker compose config

# Check resource usage
docker stats $(docker compose ps -q)

# Export all logs to file
docker compose logs -f > logs.txt

# Remove everything
docker compose down -v --rmi all --remove-orphans
```
