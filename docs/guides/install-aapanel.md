# Install aaPanel

aaPanel is a free web hosting control panel.

## Requirements

- **OS:** CentOS 7/8, Ubuntu 20.04+, Debian 11+
- **RAM:** Minimum 512MB, recommended 1GB+
- **Disk:** Minimum 10GB free
- **Ports:** 80, 443, 8888 must be free

## Installation

```bash
# Ubuntu / Debian
wget -O install.sh http://www.aapanel.com/script/install-ubuntu_6.0_en.sh
bash install.sh

# CentOS
wget -O install.sh http://www.aapanel.com/script/install_6.0_en.sh
bash install.sh
```

## Access aaPanel

```
http://<server-ip>:8888
```

Login credentials are displayed at the end of the installation. Save them immediately.

## Post-Install

### Install LAMP/LEMP Stack

1. Login to aaPanel
2. Go to **App Store**
3. Click **One-click Install**
4. Choose **Nginx + MySQL + PHP** (or Apache variant)

### Create a Website

1. Go to **Websites → Add site**
2. Enter domain, database details
3. Submit

## Security

- Change the default port `8888` in **Panel Settings → Panel Port**
- Bind panel access to specific IP if possible
- Enable firewall in **Security → Firewall**
