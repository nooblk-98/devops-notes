# Server Provisioning

## Initial Server Setup

### 1. Update System

```bash
apt update && apt upgrade -y
# or
yum update -y

# Install essentials
apt install -y curl wget git ufw fail2ban unattended-upgrades
```

### 2. Create Sudo User

```bash
adduser devops
usermod -aG sudo devops

# Copy SSH key
mkdir -p /home/devops/.ssh
cp ~/.ssh/authorized_keys /home/devops/.ssh/
chown -R devops:devops /home/devops/.ssh
chmod 700 /home/devops/.ssh
chmod 600 /home/devops/.ssh/authorized_keys
```

### 3. Harden SSH

```ini title="/etc/ssh/sshd_config"
Port 22
PermitRootLogin no
PasswordAuthentication no
PubkeyAuthentication yes
MaxAuthTries 3
ClientAliveInterval 300
ClientAliveCountMax 2
```

```bash
systemctl reload sshd
```

### 4. Configure Firewall

```bash
ufw default deny incoming
ufw default allow outgoing
ufw allow ssh
ufw allow http
ufw allow https
ufw --force enable
```

### 5. Automatic Security Updates

```bash
dpkg-reconfigure --priority=low unattended-upgrades
```

Or manually:

```ini title="/etc/apt/apt.conf.d/50unattended-upgrades"
Unattended-Upgrade::Allowed-Origins {
    "${distro_id}:${distro_codename}-security";
};
Unattended-Upgrade::AutoFixInterruptedDpkg "true";
Unattended-Upgrade::Remove-Unused-Kernel-Packages "true";
Unattended-Upgrade::Remove-New-Unused-Dependencies "true";
Unattended-Upgrade::Automatic-Reboot "false";
```

### 6. Configure Timezone & NTP

```bash
timedatectl set-timezone Asia/Colombo
timedatectl set-ntp true
timedatectl status
```

### 7. Kernel Hardening

```ini title="/etc/sysctl.d/99-hardening.conf"
# IP spoofing protection
net.ipv4.conf.all.rp_filter = 1
net.ipv4.conf.default.rp_filter = 1

# Ignore ICMP redirects
net.ipv4.conf.all.accept_redirects = 0
net.ipv6.conf.all.accept_redirects = 0

# Ignore send redirects
net.ipv4.conf.all.send_redirects = 0

# Disable source packet routing
net.ipv4.conf.all.accept_source_route = 0
net.ipv6.conf.all.accept_source_route = 0

# SYN flood protection
net.ipv4.tcp_syn_cookies = 1

# Disable ICMP ping
net.ipv4.icmp_echo_ignore_all = 1
```

```bash
sysctl -p /etc/sysctl.d/99-hardening.conf
```

### 8. Install Monitoring Agent

```bash
# Install Node Exporter
wget https://github.com/prometheus/node_exporter/releases/latest/download/node_exporter-linux-amd64.tar.gz
tar xzf node_exporter-linux-amd64.tar.gz
sudo mv node_exporter-linux-amd64/node_exporter /usr/local/bin/
```

### 9. Install Common Tools

```bash
apt install -y htop iotop net-tools dnsutils traceroute mtr \
  tcpdump ncdu rsync jq lsof neofetch
```

### 10. Verify Hardening

```bash
# Check open ports
ss -tulpn

# Check SSH config
sshd -T | grep -E "(permitrootlogin|passwordauthentication)"

# Check firewall
ufw status verbose

# Check auto-updates
systemctl status unattended-upgrades
```

## Provisioning Checklist

- [ ] System updated
- [ ] Sudo user created, root login disabled
- [ ] SSH key-only auth configured
- [ ] Firewall enabled with correct rules
- [ ] Automatic security updates configured
- [ ] Timezone and NTP set
- [ ] Kernel hardening applied
- [ ] Monitoring agent installed
- [ ] Swap configured (if needed)
- [ ] Hostname set correctly
- [ ] DNS resolvers configured
- [ ] Fail2ban installed and configured
