# Standard Firewall Rules for WordPress

## Using UFW (Ubuntu / Debian)

```bash
# Allow SSH
ufw allow ssh

# Allow HTTP and HTTPS
ufw allow 80/tcp
ufw allow 443/tcp

# Allow CloudPanel (if used)
ufw allow 8443/tcp

# Deny everything else
ufw default deny incoming
ufw default allow outgoing

# Enable
ufw --force enable
ufw status verbose
```

## Using firewalld (RHEL / Rocky / Alma)

```bash
# Allow services
firewall-cmd --permanent --add-service=ssh
firewall-cmd --permanent --add-service=http
firewall-cmd --permanent --add-service=https

# Allow CloudPanel (if used)
firewall-cmd --permanent --add-port=8443/tcp

# Reload
firewall-cmd --reload
firewall-cmd --list-all
```

## Using iptables

```bash
# Default policies
iptables -P INPUT DROP
iptables -P FORWARD DROP
iptables -P OUTPUT ACCEPT

# Allow loopback
iptables -A INPUT -i lo -j ACCEPT

# Allow established connections
iptables -A INPUT -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT

# Allow SSH, HTTP, HTTPS
iptables -A INPUT -p tcp --dport 22 -j ACCEPT
iptables -A INPUT -p tcp --dport 80 -j ACCEPT
iptables -A INPUT -p tcp --dport 443 -j ACCEPT

# Save rules
apt install -y iptables-persistent
netfilter-persistent save
```

## WP-Specific Rules

```bash
# Rate limit SSH (prevent brute force)
ufw limit ssh

# Block common WordPress attack paths
iptables -A INPUT -p tcp --dport 443 -m string --string "/wp-admin" --algo bm -m recent --set
iptables -A INPUT -p tcp --dport 443 -m string --string "/xmlrpc.php" --algo bm -m recent --set

# Allow Cloudflare IPs only for admin (if using Cloudflare)
# https://www.cloudflare.com/ips/
```

## Allow Cloudflare IPs Only (Optional)

```bash
for ip in $(curl -s https://www.cloudflare.com/ips-v4); do
    ufw allow from $ip to any port 443
done
ufw deny 443
```

!!! warning
    Only use Cloudflare-only mode if your origin server should not be directly accessible.
