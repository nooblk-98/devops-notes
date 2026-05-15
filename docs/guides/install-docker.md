# Install Docker & Docker Compose Plugin

## Ubuntu / Debian

```bash
# Remove old packages
apt remove -y docker docker-engine docker.io containerd runc

# Install dependencies
apt update
apt install -y ca-certificates curl gnupg

# Add Docker GPG key
install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | gpg --dearmor -o /etc/apt/keyrings/docker.gpg
chmod a+r /etc/apt/keyrings/docker.gpg

# Add repository
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | tee /etc/apt/sources.list.d/docker.list > /dev/null

# Install Docker Engine and Compose plugin
apt update
apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# Start and enable
systemctl enable --now docker
```

## RHEL / Rocky / Alma

```bash
# Install dependencies
yum install -y yum-utils

# Add repository
yum-config-manager --add-repo https://download.docker.com/linux/rhel/docker-ce.repo

# Install Docker Engine and Compose plugin
yum install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# Start and enable
systemctl enable --now docker
```

## Verify Installation

```bash
docker --version
docker compose version

# Test with hello-world
docker run hello-world
```

## Post-Install (Run Docker as Non-Root)

```bash
usermod -aG docker $USER
newgrp docker
```

Log out and back in for changes to take effect.
