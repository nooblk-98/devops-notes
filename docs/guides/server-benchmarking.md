# Server Benchmarking

## Tools Installation

```bash
# Web server benchmarks
apt install -y apache2-utils hey

# System benchmarks
apt install -y sysbench htop ioping

# Network
apt install -y iperf3 netperf

# Disk
apt install -y fio
```

## Web Server Benchmarks

### Apache Bench (ab)

```bash
# Basic test
ab -n 1000 -c 50 https://example.com/

# With keep-alive
ab -n 10000 -c 100 -k https://example.com/

# Test specific page
ab -n 500 -c 20 https://example.com/wp-login.php

# Output: Requests per second, Time per request, Transfer rate
```

### Hey (Modern HTTP Load)

```bash
# Simple test
hey -n 10000 -c 100 https://example.com/

# With headers
hey -n 1000 -c 50 \
  -H "Accept: text/html" \
  https://example.com/

# Output: Latency distribution, RPS, status codes
```

### wrk (Advanced)

```bash
apt install -y wrk

wrk -t12 -c400 -d30s https://example.com/
# -t: threads, -c: connections, -d: duration
```

## System Benchmarks

### CPU

```bash
# CPU benchmark
sysbench cpu --cpu-max-prime=20000 run

# Multi-threaded
sysbench cpu --cpu-max-prime=20000 --threads=4 run
```

### Memory

```bash
# Memory speed
sysbench memory --memory-block-size=1M --memory-total-size=10G run

# Memory latency
sysbench memory --memory-oper=read --memory-access-mode=rnd run
```

### Disk (fio)

```bash
# Sequential read
fio --name=seqread --ioengine=libaio --direct=1 --bs=1M --numjobs=1 \
  --size=1G --rw=read --group_reporting

# Random write
fio --name=randwrite --ioengine=libaio --direct=1 --bs=4K --numjobs=4 \
  --size=1G --rw=randwrite --group_reporting

# Mixed workload
fio --name=mixed --ioengine=libaio --direct=1 --bs=4K --numjobs=4 \
  --size=1G --rw=randrw --rwmixread=70 --group_reporting
```

### Disk Latency (ioping)

```bash
# Check latency
ioping -c 10 /

# Measure IOPS
ioping -c 100 -s 4k /

# Check disk cache
ioping -c 10 -D /
```

## Network Benchmarks

### iperf3

```bash
# On server (listener)
iperf3 -s

# On client
iperf3 -c <server-ip> -t 30

# Bidirectional
iperf3 -c <server-ip> -t 30 --bidir

# Multiple streams
iperf3 -c <server-ip> -P 4 -t 30
```

### curl Speed Test

```bash
# Download speed
curl -o /dev/null -s -w "Speed: %{speed_download} bytes/sec\n" https://example.com/100mb.bin

# Time breakdown
curl -o /dev/null -s -w "\n\
  Time to connect: %{time_connect}s\n\
  Time to first byte: %{time_starttransfer}s\n\
  Total time: %{time_total}s\n\
  Download speed: %{speed_download} bytes/sec\n" \
  https://example.com/
```

## PHP Benchmarks

```bash
# PHP speed test
php -r "\$start = microtime(true);
for(\$i = 0; \$i < 100000; \$i++) { \$a = sqrt(\$i); }
echo 'PHP Time: ' . (microtime(true) - \$start) . 's\n';"

# Opcache status
php -r "print_r(opcache_get_status());"
```

## MySQL Benchmarks

```bash
# Built-in benchmark
mysqlslap --user=root --password --concurrency=50 --iterations=10 \
  --query="SELECT * FROM wp_posts LIMIT 1000;" \
  --create-schema=wordpress

# sysbench MySQL
sysbench /usr/share/sysbench/oltp_read_write.lua \
  --mysql-user=root --mysql-password=pass \
  --mysql-db=wordpress --tables=10 --table-size=100000 \
  --threads=4 --time=60 run
```

## Full Server Score

```bash title="/usr/local/bin/benchmark.sh"
#!/bin/bash
echo "=== CPU ==="
sysbench cpu --cpu-max-prime=20000 --threads=$(nproc) run | grep "events per second"

echo "=== Memory ==="
sysbench memory --memory-block-size=1M --memory-total-size=10G run | grep "transferred"

echo "=== Disk Read ==="
fio --name=read --ioengine=libaio --direct=1 --bs=1M --size=1G --rw=read \
  --group_reporting --output-format=json 2>/dev/null | python3 -c "import sys,json; d=json.load(sys.stdin); print(f'Read: {d[\"jobs\"][0][\"read\"][\"bw_bytes\"]/1024/1024:.0f} MB/s')"

echo "=== Disk Write ==="
fio --name=write --ioengine=libaio --direct=1 --bs=1M --size=1G --rw=write \
  --group_reporting --output-format=json 2>/dev/null | python3 -c "import sys,json; d=json.load(sys.stdin); print(f'Write: {d[\"jobs\"][0][\"write\"][\"bw_bytes\"]/1024/1024:.0f} MB/s')"

echo "=== Network (loopback) ==="
iperf3 -c 127.0.0.1 -t 10 2>/dev/null | grep "sender"
```

## Monitoring During Load Test

```bash
# Real-time system stats
htop

# Network
nload
iftop

# Disk IO
iotop

# Memory
free -h -s 2
```

## When to Benchmark

| When | What to Test |
|------|-------------|
| New server provisioned | CPU, disk, network baseline |
| Before launch | Full web server load test |
| After config changes | Compare before/after |
| Performance degradation | CPU, memory, disk IO |
| Monthly | Quick system health check |

## Benchmark Checklist

- [ ] Baseline scores recorded for comparison
- [ ] Tested during off-peak hours
- [ ] Multiple test runs (3+) for consistency
- [ ] Network bandwidth considered
- [ ] Resource limits (CPU, memory) noted
- [ ] Results documented in server inventory
