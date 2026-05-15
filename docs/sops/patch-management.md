# Patch Management SOP

**Version:** 1.0  
**Owner:** DevOps Team

## Purpose

Establish a consistent process for applying security patches, OS updates, and kernel upgrades.

## Schedule

| Environment | Frequency | Window | Approval |
|-------------|-----------|--------|----------|
| Development | Weekly | Any | Auto |
| Staging | Bi-weekly | Business hours | Team lead |
| Production | Monthly | Scheduled maintenance window | Change request |

## Procedure

### 1. Pre-Patch Checklist

- [ ] Backup taken (database + files)
- [ ] Snapshot created (if cloud VM)
- [ ] Maintenance window communicated
- [ ] Rollback plan ready
- [ ] Staging patched and verified first

### 2. Update Package Lists

```bash
# Debian / Ubuntu
apt update

# RHEL / Rocky / Alma
yum check-update
```

### 3. Review Available Updates

```bash
# See upgradable packages
apt list --upgradable

# Check for kernel updates
apt list --upgradable 2>/dev/null | grep linux-image

# Check security updates only
apt list --upgradable 2>/dev/null | grep -i security
```

### 4. Apply Updates

```bash
# Apply all updates (non-interactive)
apt upgrade -y

# Security-only updates
apt upgrade -y $(apt list --upgradable 2>/dev/null | grep -i security | cut -d/ -f1)

# Kernel only
apt install -y linux-image-generic
```

### 5. Check for Reboot Required

```bash
# Check if reboot needed
cat /var/run/reboot-required 2>/dev/null && echo "REBOOT REQUIRED"

# Check kernel version vs running
uname -r
dpkg -l | grep linux-image-$(uname -r)

# Check with needs-restarting (RHEL)
needs-restarting
```

### 6. Reboot Procedure

```bash
# Schedule reboot with warning
shutdown -r +5 "System patching - reboot in 5 minutes"

# Or immediate
systemctl reboot
```

### 7. Post-Patch Verification

```bash
# Verify services are running
systemctl status nginx mysql php8.2-fpm

# Verify site loads
curl -I https://example.com

# Check for errors
journalctl -p err -b

# Verify kernel updated
uname -r
```

## Kernel Upgrade (Major)

```bash
# Install new kernel
apt install -y linux-image-generic linux-headers-generic

# Reboot
systemctl reboot

# Verify new kernel booted
uname -r

# Remove old kernels
apt autoremove --purge
```

## Emergency Patching

For critical CVEs (CVSS >= 9.0):

1. Patch immediately regardless of schedule
2. Notify stakeholders
3. Apply to staging first (30 min max)
4. Deploy to production
5. Postmortem within 24 hours

## Rollback

```bash
# Rollback packages (apt)
apt install -y <package>=<previous-version>

# Boot from old kernel (grub)
# Select previous kernel in GRUB menu at boot

# Restore snapshot via hypervisor
```

## Automation

```bash title="/etc/cron.weekly/patch-security"
#!/bin/bash
apt update && apt upgrade -y $(apt list --upgradable 2>/dev/null | grep -i security | cut -d/ -f1)
echo "$(date): Security patches applied" >> /var/log/patch-management.log
```

## Verification

- [ ] All servers patched within schedule
- [ ] Staging patched and verified before production
- [ ] Services healthy post-reboot
- [ ] Old kernels removed
- [ ] Rollback plan available
