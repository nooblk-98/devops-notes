# Jenkins Installation

Install and configure Jenkins server for CI/CD automation. Covers both Docker and manual installation on Ubuntu/Debian and RHEL-based systems.

## Prerequisites

- Linux server (Ubuntu 22.04+ / Debian 12+ / Rocky Linux 9+)
- 2 GB RAM minimum, 4 GB recommended
- 20 GB disk space
- Root or sudo access
- Port 8080 accessible (Jenkins web UI)

## Option 1: Manual Installation

### Install Java

Jenkins requires Java 17 or 21 (OpenJDK recommended).

**Ubuntu / Debian:**

```bash
apt update
apt install -y openjdk-17-jdk-headless
java -version
```

**RHEL / Rocky / Alma:**

```bash
dnf install -y java-17-openjdk-headless
java -version
```

Set `JAVA_HOME`:

```bash
echo "JAVA_HOME=$(dirname $(dirname $(readlink -f $(which java))))" | tee -a /etc/environment
source /etc/environment
```

### Add Jenkins Repository

**Ubuntu / Debian:**

```bash
curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key | tee /usr/share/keyrings/jenkins-keyring.asc > /dev/null
echo "deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] https://pkg.jenkins.io/debian-stable binary/" | tee /etc/apt/sources.list.d/jenkins.list > /dev/null
apt update
```

**RHEL / Rocky / Alma:**

```bash
curl -fsSL https://pkg.jenkins.io/redhat-stable/jenkins.repo -o /etc/yum.repos.d/jenkins.repo
rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io-2023.key
dnf upgrade
```

### Install Jenkins

**Ubuntu / Debian:**

```bash
apt install -y jenkins
systemctl enable --now jenkins
```

**RHEL / Rocky / Alma:**

```bash
dnf install -y jenkins
systemctl enable --now jenkins
```

### Firewall

```bash
ufw allow 8080/tcp
# or
firewall-cmd --permanent --add-port=8080/tcp
firewall-cmd --reload
```

### Access Jenkins

```bash
# Get initial admin password
cat /var/lib/jenkins/secrets/initialAdminPassword
```

Open `http://<your-server-ip>:8080` in a browser, enter the password, and follow the setup wizard (install suggested plugins + create admin user).

### Jenkins Data Directory

Default location: `/var/lib/jenkins/`

Key subdirectories:

| Path | Purpose |
|------|---------|
| `jobs/` | Build job configurations |
| `workspace/` | Build workspace directories |
| `plugins/` | Installed plugins |
| `secrets/` | Credentials and keys |
| `users/` | User configurations |
| `logs/` | Jenkins logs |

## Option 2: Docker Installation

### Quick Start

```bash
docker run -d \
  --name jenkins \
  --restart unless-stopped \
  -p 8080:8080 \
  -p 50000:50000 \
  -v jenkins_home:/var/jenkins_home \
  jenkins/jenkins:lts-jdk17
```

### With Docker Compose

Create `compose.yml`:

```yaml
services:
  jenkins:
    image: jenkins/jenkins:lts-jdk17
    container_name: jenkins
    restart: unless-stopped
    ports:
      - "8080:8080"
      - "50000:50000"
    volumes:
      - jenkins_home:/var/jenkins_home
      - /var/run/docker.sock:/var/run/docker.sock
    environment:
      - JAVA_OPTS=-Djenkins.install.runSetupWizard=false

volumes:
  jenkins_home:
```

Start:

```bash
docker compose up -d
```

### Enable Docker-in-Docker (Pipeline Usage)

To run Docker commands inside Jenkins pipelines:

```bash
docker exec -u root jenkins apt update
docker exec -u root jenkins apt install -y docker.io
```

Then add `jenkins` user to docker group:

```bash
docker exec -u root jenkins usermod -aG docker jenkins
```

### Backup Jenkins

```bash
# Docker
docker run --rm -v jenkins_home:/data -v $(pwd):/backup alpine tar czf /backup/jenkins-backup-$(date +%Y%m%d).tar.gz -C /data .

# Manual
tar czf jenkins-backup-$(date +%Y%m%d).tar.gz /var/lib/jenkins/
```

## Post-Installation

### Install Plugins via CLI

```bash
# List installed plugins
java -jar jenkins-cli.jar -s http://localhost:8080/ list-plugins

# Install a plugin
java -jar jenkins-cli.jar -s http://localhost:8080/ install-plugin <plugin-name>
```

### Nginx Reverse Proxy (optional)

```nginx
server {
    listen 80;
    server_name jenkins.example.com;

    location / {
        proxy_pass http://127.0.0.1:8080;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

### Common Plugins

| Plugin | Purpose |
|--------|---------|
| Blue Ocean | Modern pipeline UI |
| Pipeline | Pipeline as Code |
| Docker Pipeline | Docker build/push in pipelines |
| Git | Git integration |
| GitHub Integration | GitHub webhook triggers |
| Slack Notification | Build notifications |
| SonarQube Scanner | Code quality analysis |

## Troubleshooting

**Jenkins fails to start:**
```bash
journalctl -u jenkins -n 50
```

**Disk space full:**
```bash
du -sh /var/lib/jenkins/workspace/*
rm -rf /var/lib/jenkins/workspace/old-job
```

**Permission denied for Docker socket:**
```bash
usermod -aG docker jenkins
systemctl restart jenkins
```
