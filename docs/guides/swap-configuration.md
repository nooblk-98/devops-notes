# Swap Configuration

## When to Use Swap

| RAM Size | Swap Size | Recommendation |
|----------|-----------|----------------|
| < 2GB | 2 × RAM | Essential for low-memory servers |
| 2-4GB | 1 × RAM | Good safety net |
| 4-8GB | 4GB | Adequate |
| 8-16GB | 2-4GB | Minimal |
| > 16GB | 2GB | Rarely needed |

## Check Current Swap

```bash
# Check if swap is active
swapon --show

# Check memory & swap usage
free -h

# Check filesystem for swap file
ls -lh /swapfile
```

## Create Swap File

```bash
# 1. Create swap file (4GB example)
fallocate -l 4G /swapfile
# OR (if fallocate not available)
dd if=/dev/zero of=/swapfile bs=1M count=4096

# 2. Set correct permissions
chmod 600 /swapfile

# 3. Format as swap
mkswap /swapfile

# 4. Enable swap
swapon /swapfile

# 5. Verify
swapon --show
free -h
```

## Make Swap Permanent

```bash
# Add to /etc/fstab
echo '/swapfile none swap sw 0 0' | tee -a /etc/fstab

# Verify fstab entry
cat /etc/fstab | grep swap
```

## Swap Swappiness

Swappiness controls how aggressively the kernel uses swap (0-100).

```bash
# Check current value
cat /proc/sys/vm/swappiness

# Recommended values:
# 10 — Good for servers (less swap usage)
# 60 — Default (balance)
# 100 — Aggressive swap

# Set temporarily
sysctl vm.swappiness=10

# Make permanent
echo 'vm.swappiness=10' | tee -a /etc/sysctl.conf
```

## Remove Swap

```bash
# 1. Disable swap
swapoff /swapfile

# 2. Remove from fstab
sed -i '/swapfile/d' /etc/fstab

# 3. Delete file
rm /swapfile

# 4. Verify
swapon --show
free -h
```

## Swap for Docker

Docker containers can also use swap. Set limits:

```yaml title="docker-compose.yml"
services:
  app:
    # Memory + swap limit
    mem_limit: 512m
    memswap_limit: 1g  # 512MB RAM + 512MB swap
```

## Monitoring Swap Usage

```bash
# Real-time swap usage
free -h -s 5

# Processes using swap
for pid in /proc/*/status; do
    awk '/VmSwap|Name/{printf "%s ", $2} END{print ""}' $pid 2>/dev/null
done | sort -k2 -n -r | head -10

# Swap usage per process
top -o %MEM
# Press 'c' for command, check SWAP column

# Or use smem
apt install -y smem
smem -s swap -r
```

## Swap on SSD vs HDD

| Drive Type | Notes |
|------------|-------|
| **SSD** | Fast swap, but wear from writes. Reduce swappiness to 10. |
| **HDD** | Slow swap, avoid if possible. Add more RAM instead. |
| **NVMe** | Excellent swap performance. Still reduce swappiness. |

## Verdict

```bash
# Quick check script
echo "=== Swap Status ==="
swapon --show
echo ""
echo "=== Memory ==="
free -h
echo ""
echo "=== Swappiness ==="
cat /proc/sys/vm/swappiness
echo ""
echo "=== Swap Usage by Top Processes ==="
for pid in /proc/*/status; do
    awk '/VmSwap|Name/{printf "%s ", $2} END{print ""}' $pid 2>/dev/null
done | sort -k2 -n -r | head -5
```
