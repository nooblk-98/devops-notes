---
tags:
  - dns
  - cloudflare
  - domain
---

# DNS Management

**Version:** 1.0  
**Owner:** DevOps Team

## Purpose

Standardize DNS record management, Cloudflare integration, and troubleshooting.

## Common DNS Record Types

| Type | Purpose | Example |
|------|---------|---------|
| A | Maps domain to IPv4 | `example.com → 192.0.2.1` |
| AAAA | Maps domain to IPv6 | `example.com → 2001:db8::1` |
| CNAME | Maps domain to another domain | `www → example.com` |
| MX | Mail server | `example.com → mail.example.com` |
| TXT | Text data (verification, SPF, DKIM) | `SPF "v=spf1 include:_spf.google.com ~all"` |
| NS | Name servers | `example.com → ns1.cloudflare.com` |
| SRV | Service location | `_sip._tcp.example.com` |

## Adding DNS Records

### Via Cloudflare Dashboard

1. Login to [Cloudflare Dashboard](https://dash.cloudflare.com)
2. Select domain
3. Go to **DNS → Records**
4. Click **Add Record**
5. Fill type, name, value, TTL
6. Toggle proxy (orange cloud = proxied, gray = DNS only)

### Via Cloudflare API

```bash
# Add A record
curl -s -X POST "https://api.cloudflare.com/client/v4/zones/$ZONE_ID/dns_records" \
  -H "Authorization: Bearer $API_TOKEN" \
  -H "Content-Type: application/json" \
  --data '{
    "type": "A",
    "name": "www",
    "content": "192.0.2.1",
    "ttl": 300,
    "proxied": true
  }'

# List records
curl -s "https://api.cloudflare.com/client/v4/zones/$ZONE_ID/dns_records" \
  -H "Authorization: Bearer $API_TOKEN" | jq '.result[] | {name, type, content}'
```

## Cloudflare Configuration

### Recommended Settings

| Setting | Value | Reason |
|---------|-------|--------|
| SSL/TLS | Full (strict) | End-to-end encryption |
| Always Use HTTPS | On | Redirect HTTP to HTTPS |
| Min TLS Version | 1.2 | Security |
| Auto Minify | JS, CSS, HTML | Performance |
| Brotli | On | Compression |
| HTTP/2 | On | Performance |
| IPv6 | On | Compatibility |
| Security Level | Medium | Balance |
| Bot Fight Mode | On | Block bots |
| WAF | On | Web application firewall |

### Page Rules

```yaml
# Force HTTPS
example.com/*:
  - SSL: Full
  - Always Use HTTPS: On

# Cache static assets
example.com/wp-content/*:
  - Cache Level: Cache Everything
  - Edge Cache TTL: 7 days

# Bypass cache for admin
example.com/wp-admin/*:
  - Cache Level: Bypass
  - Disable Performance: On
```

## Cloudflare + Nginx (Origin Server)

Get real visitor IP:

```nginx title="/etc/nginx/conf.d/cloudflare.conf"
set_real_ip_from 173.245.48.0/20;
set_real_ip_from 103.21.244.0/22;
# ... all Cloudflare IPs
real_ip_header CF-Connecting-IP;
```

```bash
# Get latest Cloudflare IPs
curl -s https://www.cloudflare.com/ips-v4
curl -s https://www.cloudflare.com/ips-v6
```

## DNSSEC

### Enable on Cloudflare

1. Dashboard → Domain → **DNS → DNSSEC**
2. Click **Enable DNSSEC**
3. Copy DS record details

### Add at Registrar

1. Go to domain registrar
2. Find DNSSEC settings
3. Add the DS record provided by Cloudflare

## Troubleshooting

### Check DNS Propagation

```bash
nslookup example.com
dig example.com +short

# Query specific nameserver
dig @ns1.cloudflare.com example.com

# Check propagation globally
# https://www.whatsmydns.net/
```

### Common Issues

| Symptom | Cause | Fix |
|---------|-------|-----|
| Site not loading | DNS not propagated | Wait 5-30 min (TTL) |
| SSL errors | SSL/TLS setting wrong | Set to Full (strict) |
| Email not sending | Missing MX/SPF records | Add MX and SPF TXT record |
| Subdomain not working | Missing A/CNAME record | Add record for subdomain |
| Cloudflare 521 | Origin server down | Check web server on origin |

### Flush DNS Cache

```bash
# Local
sudo systemd-resolve --flush-caches

# Cloudflare (purge cache)
curl -X POST "https://api.cloudflare.com/client/v4/zones/$ZONE_ID/purge_cache" \
  -H "Authorization: Bearer $API_TOKEN" \
  -H "Content-Type: application/json" \
  --data '{"purge_everything": true}'
```

## Verification

- [ ] All required A/CNAME records exist
- [ ] MX records configured for email
- [ ] SPF, DKIM, DMARC records set
- [ ] SSL/TLS set to Full (strict)
- [ ] DNSSEC enabled
- [ ] Page rules configured for caching
- [ ] Origin server IP not exposed
- [ ] WAF enabled
