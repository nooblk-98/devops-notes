# SSL Troubleshooting

## Common SSL Errors

| Error | Cause | Fix |
|-------|-------|-----|
| `NET::ERR_CERT_COMMON_NAME_INVALID` | Domain doesn't match certificate | Re-issue cert with correct domain |
| `NET::ERR_CERT_DATE_INVALID` | Certificate expired | Renew certificate |
| `NET::ERR_CERT_AUTHORITY_INVALID` | Untrusted issuer | Install intermediate cert |
| `SSL: error:0A000086` | Wrong key/cert pair | Verify key matches cert |
| `ssl_stapling` errors | OCSP responder unreachable | Check OCSP URL in cert |

## Check Certificate Details

```bash
# View cert info
openssl x509 -in /etc/letsencrypt/live/example.com/fullchain.pem -text -noout

# Check expiry
openssl x509 -in /etc/letsencrypt/live/example.com/fullchain.pem -noout -enddate

# Check subject/issuer
openssl x509 -in /etc/letsencrypt/live/example.com/fullchain.pem -noout -subject -issuer

# Check domains (SAN)
openssl x509 -in /etc/letsencrypt/live/example.com/fullchain.pem -noout -ext subjectAltName
```

## Verify Key Matches Certificate

```bash
# Compare modulus (must match)
openssl x509 -noout -modulus -in /etc/letsencrypt/live/example.com/fullchain.pem | openssl md5
openssl rsa -noout -modulus -in /etc/letsencrypt/live/example.com/privkey.pem | openssl md5
```

## Check Certificate Chain

```bash
# View full chain
openssl crl2pkcs7 -nocrl -certfile /etc/letsencrypt/live/example.com/fullchain.pem | openssl pkcs7 -print_certs -text -noout | grep "Subject:\|Issuer:"

# Verify chain
openssl verify -CAfile /etc/ssl/certs/ca-certificates.crt /etc/letsencrypt/live/example.com/fullchain.pem
```

## Test SSL from Outside

```bash
# Using openssl
openssl s_client -connect example.com:443 -servername example.com

# Test with specific protocol
openssl s_client -connect example.com:443 -tls1_2
openssl s_client -connect example.com:443 -tls1_3

# Using curl
curl -vI https://example.com 2>&1 | grep -i "ssl\|certificate"

# Using ssllabs (online)
# https://www.ssllabs.com/ssltest/analyze.html?d=example.com
```

## Renew Certificate

```bash
# Dry run
certbot renew --dry-run

# Force renew
certbot renew --force-renewal

# Renew specific domain
certbot certonly --force-renewal -d example.com

# Reload web server
nginx -t && systemctl reload nginx
```

## Install Intermediate Certificate

Nginx config must include full chain:

```nginx
ssl_certificate /etc/letsencrypt/live/example.com/fullchain.pem;     # Cert + intermediates
ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem;    # Private key
```

`fullchain.pem` = server cert + intermediates.  
NEVER use `cert.pem` alone (browsers will show untrusted).

## Mixed Content Issues

HTTPS page loading HTTP resources:

```bash
# Find mixed content
curl -s https://example.com | grep -i "http://"

# Fix with WP-CLI
wp search-replace "http://example.com" "https://example.com" --all-tables

# Or force HTTPS in wp-config
define('FORCE_SSL_ADMIN', true);

# Cloudflare automatic HTTPS rewrites
# Dashboard → SSL/TLS → Edge Certificates → Automatic HTTPS Rewrites → ON
```

## Cloudflare SSL Settings

| Mode | Description |
|------|-------------|
| Off | No encryption |
| Flexible | Browser→CF encrypted, CF→server plain |
| Full | End-to-end encrypted, but CF trusts any origin cert |
| **Full (strict)** | End-to-end encrypted, CF validates origin cert |

**Recommended:** `Full (strict)`

## Check OCSP Stapling

```bash
# Test OCSP
openssl ocsp -issuer /etc/letsencrypt/live/example.com/chain.pem \
  -cert /etc/letsencrypt/live/example.com/cert.pem \
  -url $(openssl x509 -in /etc/letsencrypt/live/example.com/cert.pem -noout -ocsp_uri) \
  -header "Host" $(openssl x509 -in /etc/letsencrypt/live/example.com/cert.pem -noout -ocsp_uri | awk -F/ '{print $3}')

# Check Nginx OCSP stapling
openssl s_client -connect example.com:443 -tls1_2 -servername example.com -status 2>&1 | grep -i "OCSP response"
```

## Certificate Revocation

```bash
# If private key compromised
certbot revoke --cert-path /etc/letsencrypt/live/example.com/cert.pem

# Generate new cert
certbot certonly -d example.com
```

## Quick Diagnostic Script

```bash
#!/bin/bash
DOMAIN="example.com"

echo "=== Certificate Expiry ==="
openssl x509 -in /etc/letsencrypt/live/$DOMAIN/fullchain.pem -noout -enddate

echo "=== Subject ==="
openssl x509 -in /etc/letsencrypt/live/$DOMAIN/fullchain.pem -noout -subject

echo "=== SAN Domains ==="
openssl x509 -in /etc/letsencrypt/live/$DOMAIN/fullchain.pem -noout -ext subjectAltName | grep DNS

echo "=== SSL Test ==="
echo | openssl s_client -connect $DOMAIN:443 -servername $DOMAIN 2>/dev/null | openssl x509 -noout -dates

echo "=== Mixed Content Check ==="
curl -s https://$DOMAIN | grep -c "http://"

echo "=== HTTP Status ==="
curl -o /dev/null -s -w "%{http_code}" https://$DOMAIN
```
