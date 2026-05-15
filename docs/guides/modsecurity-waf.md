# WAF Rules (ModSecurity)

## Installation

### Ubuntu / Debian (Nginx)

ModSecurity as a standalone or Nginx module:

```bash
# Install libmodsecurity3
apt install -y libmodsecurity3

# Or compile with Nginx (dynamic module)
git clone --depth 1 https://github.com/SpiderLabs/ModSecurity-nginx.git
```

### Docker with ModSecurity

```yaml title="docker-compose.yml"
services:
  nginx:
    image: owasp/modsecurity-crs:nginx
    container_name: waf
    restart: unless-stopped
    ports:
      - "80:80"
      - "443:443"
    environment:
      PARANOIA: 2
      BACKEND: http://app:3000
      PORT: 80
    volumes:
      - ./nginx/custom.conf:/etc/nginx/conf.d/custom.conf
```

## OWASP Core Rule Set (CRS)

The OWASP CRS is the standard set of WAF rules.

### Manual Install

```bash
# Download CRS
git clone https://github.com/coreruleset/coreruleset /etc/modsecurity/crs

# Copy example config
cp /etc/modsecurity/crs/crs-setup.conf.example /etc/modsecurity/crs/crs-setup.conf
```

### Enable in ModSecurity

```ini title="/etc/modsecurity/modsecurity.conf"
SecRuleEngine On

# Include CRS
Include /etc/modsecurity/crs/crs-setup.conf
Include /etc/modsecurity/crs/rules/*.conf
```

## Rule Categories

| Category | Rules | Protection |
|----------|-------|------------|
| `REQUEST-901-INITIALIZATION` | Initialization | Setup |
| `REQUEST-903.9001-DRUPAL-EXCLUSION` | Drupal exclusions | Reduce false positives |
| `REQUEST-903.9002-WORDPRESS-EXCLUSION` | WordPress exclusions | Reduce false positives |
| `REQUEST-905-COMMON-EXCEPTIONS` | Common exceptions | Plugin/file exclusions |
| `REQUEST-910-IP-REPUTATION` | IP reputation | Block malicious IPs |
| `REQUEST-911-METHOD-ENFORCEMENT` | HTTP methods | Block invalid methods |
| `REQUEST-912-DOS-PROTECTION` | DoS protection | Rate limiting |
| `REQUEST-913-SCANNER-DETECTION` | Scanner detection | Block security scanners |
| `REQUEST-920-PROTOCOL-ENFORCEMENT` | Protocol enforcement | HTTP validation |
| `REQUEST-921-PROTOCOL-ATTACK` | Protocol attack | Protocol abuse detection |
| `REQUEST-930-APPLICATION-ATTACK-LFI` | LFI | Local file inclusion |
| `REQUEST-931-APPLICATION-ATTACK-RFI` | RFI | Remote file inclusion |
| `REQUEST-932-APPLICATION-ATTACK-RCE` | RCE | Remote code execution |
| `REQUEST-933-APPLICATION-ATTACK-PHP` | PHP injection | PHP-specific attacks |
| `REQUEST-934-APPLICATION-ATTACK-NODEJS` | Node.js injection | Node.js attacks |
| `REQUEST-941-APPLICATION-ATTACK-XSS` | XSS | Cross-site scripting |
| `REQUEST-942-APPLICATION-ATTACK-SQLI` | SQLi | SQL injection |
| `REQUEST-943-APPLICATION-ATTACK-SESSION-FIXATION` | Session | Session fixation |
| `REQUEST-944-APPLICATION-ATTACK-JAVA` | Java | Java attacks |
| `REQUEST-949-BLOCKING-EVALUATION` | Blocking | Final blocking eval |
| `RESPONSE-950-DATA-LEAKAGES` | Data leak | Information disclosure |
| `RESPONSE-951-DATA-LEAKAGES-SQL` | SQL leak | SQL errors |
| `RESPONSE-952-DATA-LEAKAGES-JAVA` | Java leak | Java errors |
| `RESPONSE-953-DATA-LEAKAGES-PHP` | PHP leak | PHP errors |
| `RESPONSE-954-DATA-LEAKAGES-IIS` | IIS leak | IIS errors |
| `RESPONSE-959-BLOCKING-EVALUATION` | Blocking | Response blocking eval |
| `RESPONSE-980-CORRELATION` | Correlation | Alert correlation |

## Paranoia Levels

| Level | Description | False Positives |
|-------|-------------|-----------------|
| 1 | Basic rules (default) | Low |
| 2 | Advanced rules | Medium |
| 3 | Aggressive rules | High |
| 4 | Extreme rules | Very high |

```bash
# Set paranoia level
PARANOIA=2
```

## Custom Rules

```apache title="/etc/modsecurity/custom-rules.conf"
# Block specific IP
SecRule REMOTE_ADDR "^123\.45\.67\.89$" "id:1000,phase:1,deny,msg:'Blocked malicious IP'"

# Block user-agent
SecRule REQUEST_HEADERS:User-Agent "nikto|sqlmap" "id:1001,phase:1,deny,msg:'Blocked malicious scanner'"

# Block wp-login brute force (rate limit)
SecRule IP:BRUTE_FORCE_BLOCK "@streq 1" "id:1002,phase:1,deny,msg:'Brute force blocked'"

SecAction "id:1003,phase:5,initcol:ip=%{REMOTE_ADDR},pass,nolog"

SecRule REQUEST_FILENAME "/wp-login\.php" "id:1004,phase:2,t:none,setvar:ip.bf_counter=+1,expirevar:ip.bf_counter=60,pass,nolog"

SecRule IP:BF_COUNTER "@gt 5" "id:1005,phase:2,t:none,setvar:ip.brute_force_block=1,expirevar:ip.brute_force_block=300,msg:'Brute force detected'"

# Whitelist Cloudflare IPs
SecRule REMOTE_ADDR "@ipMatch 173.245.48.0/20,103.21.244.0/22" "id:1006,phase:1,allow,msg:'Cloudflare whitelist'"
```

## ModSecurity + Nginx

```nginx
server {
    # Enable ModSecurity
    modsecurity on;
    modsecurity_rules_file /etc/modsecurity/modsecurity.conf;

    location / {
        proxy_pass http://127.0.0.1:3000;
    }
}
```

## Logging & Monitoring

```bash
# Audit log
tail -f /var/log/modsec_audit.log

# Check for blocked requests
grep "Blocked" /var/log/modsec_audit.log

# Count by rule ID
grep -oP 'id:"\K[^"]+' /var/log/modsec_audit.log | sort | uniq -c | sort -rn
```

## Tuning (Reduce False Positives)

### Add Exclusions

```apache
# WordPress admin exclusion
SecRule REQUEST_FILENAME "@beginsWith /wp-admin" \
    "id:2000,phase:1,pass,nolog,ctl:ruleEngine=Off"
```

### Change to Detection-Only

```ini
SecRuleEngine DetectionOnly
```

### Whitelist Specific Parameters

```apache
# Allow HTML in post_content
SecRuleUpdateTargetById 942100 "!ARGS:post_content"
SecRuleUpdateTargetById 942110 "!ARGS:post_content"
```

## Test Your WAF

```bash
# Test SQL injection
curl "https://example.com/?id=1' OR '1'='1"

# Test XSS
curl "https://example.com/?q=<script>alert(1)</script>"

# Test LFI
curl "https://example.com/?file=../../../etc/passwd"

# Should all return 403 Forbidden
```

## Verification

- [ ] ModSecurity module loaded (`nginx -V 2>&1 | grep modsecurity`)
- [ ] OWASP CRS installed and enabled
- [ ] Paranoia level set (1-4)
- [ ] Audit logging enabled
- [ ] WordPress exclusions configured (if needed)
- [ ] False positives reviewed and tuned
- [ ] Test attacks blocked (403)
- [ ] Mode: DetectionOnly initially, then switched to On
