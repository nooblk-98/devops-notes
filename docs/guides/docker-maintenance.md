# Docker Maintenance

## Cleanup

### Remove Unused Resources

```bash
# Remove stopped containers
docker container prune

# Remove unused images
docker image prune -a

# Remove unused volumes
docker volume prune

# Remove unused networks
docker network prune

# Remove everything unused (containers, images, volumes, networks)
docker system prune -a --volumes
```

### Clean Periodic (Cron)

```bash title="/etc/cron.daily/docker-cleanup"
#!/bin/bash
docker system prune -a --volumes -f 2>&1 | logger -t docker-cleanup
```

```bash
chmod +x /etc/cron.daily/docker-cleanup
```

## Disk Usage

```bash
# Show disk usage by Docker objects
docker system df

# Verbose
docker system df -v

# Check container log sizes
du -sh /var/lib/docker/containers/*/*-json.log

# Check volume sizes
du -sh /var/lib/docker/volumes/*
```

## Log Management

```yaml title="docker-compose.yml"
services:
  app:
    logging:
      driver: json-file
      options:
        max-size: "10m"
        max-file: "3"
```

Global limit in `/etc/docker/daemon.json`:

```json
{
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "10m",
    "max-file": "3"
  }
}
```

```bash
systemctl restart docker
```

## Backup & Restore Volumes

### Backup

```bash
docker run --rm \
  -v my_volume:/source \
  -v $(pwd):/backup \
  alpine tar czf /backup/my_volume-$(date +%Y%m%d).tar.gz -C /source .
```

### Restore

```bash
docker run --rm \
  -v my_volume:/target \
  -v $(pwd):/backup \
  alpine tar xzf /backup/my_volume-20240101.tar.gz -C /target
```

## Image Management

```bash
# List images by size
docker images --format "{{.Repository}}:{{.Tag}} {{.Size}}" | sort -k2 -h

# Remove dangling images
docker image prune -f

# Remove images older than 30 days
docker image prune -a --filter "until=720h" -f

# Export/import images
docker save myapp:latest | gzip > myapp-latest.tar.gz
gunzip -c myapp-latest.tar.gz | docker load
```

## Container Auto-Restart

```yaml title="docker-compose.yml"
services:
  app:
    restart: unless-stopped
```

Policies: `no`, `always`, `on-failure`, `unless-stopped`

## Update Containers

```bash
# Pull and recreate
docker compose pull
docker compose up -d --force-recreate

# With zero-downtime (if configured)
docker compose up -d --no-deps --build <service>
```

## Monitor Docker

```bash
# Real-time stats
docker stats

# Resource usage per container
docker stats --no-stream --format "table {{.Name}}\t{{.CPUPerc}}\t{{.MemUsage}}"

# Container events
docker events --filter 'type=container' --filter 'event=die'

# Check Docker daemon health
docker info
docker system events --since 5m
```

## Security

```bash
# Scan images for vulnerabilities
docker scan <image>

# Run container with read-only root
docker run --read-only --tmpfs /tmp <image>

# Limit resources
docker run --memory=512m --cpus=0.5 <image>

# Use non-root user
docker run -u 1000:1000 <image>
```

## Docker Registry Management

```bash
# Login to registry
docker login registry.example.com

# Tag and push
docker tag myapp:latest registry.example.com/myapp:latest
docker push registry.example.com/myapp:latest

# Pull and run
docker pull registry.example.com/myapp:latest
```

## Verification

- [ ] Disk usage checked weekly
- [ ] Unused images cleaned monthly
- [ ] Log rotation configured
- [ ] Volumes backed up
- [ ] Containers restart policy set
- [ ] Resource limits applied
