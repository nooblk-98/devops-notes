# Install CloudPanel

CloudPanel is a free control panel for PHP applications with built-in security hardening.

## Requirements

- **OS:** Ubuntu 22.04 / 24.04 LTS (Debian 12 supported)
- **RAM:** Minimum 1GB, recommended 2GB+
- **Disk:** Minimum 10GB free
- **Ports:** 80, 443, 8443 must be free

## Installation

```bash
# Download and run installer
curl -sS https://installer.cloudpanel.io/ce/v2/install.sh -o install.sh
sudo bash install.sh
```

The installer will:
- Install Nginx, MySQL/MariaDB, PHP
- Set up CloudPanel web interface
- Configure firewall (UFW)

## Access CloudPanel

```
https://<server-ip>:8443
```

Login credentials are shown at the end of the installation script.

## Create a WordPress Site

1. Login to CloudPanel at `https://<server-ip>:8443`
2. Go to **Websites → Create Website**
3. Select **PHP** → **WordPress**
4. Enter domain name, database details, and admin user
5. Click **Create**

## Useful Commands

```bash
# CloudPanel CLI
clpctl --help

# List sites
clpctl site:list

# Create database
clpctl db:create --databaseName=mydb --databaseUserName=myuser --databaseUserPassword=mypass
```
