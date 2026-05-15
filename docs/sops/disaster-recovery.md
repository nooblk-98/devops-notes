# Disaster Recovery SOP

**Version:** 1.0  
**Owner:** DevOps Team

## Purpose

Define the disaster recovery plan including RTO/RPO, failover procedures, and recovery drills.

## Definitions

| Term | Meaning |
|------|---------|
| **RTO** | Recovery Time Objective — max acceptable downtime |
| **RPO** | Recovery Point Objective — max acceptable data loss |
| **DR** | Disaster Recovery |
| **HA** | High Availability |
| **MTTR** | Mean Time to Recover |

## Objectives

| Tier | RTO | RPO | Example |
|------|-----|-----|---------|
| Critical | < 1 hour | < 15 min | Production websites, payment |
| High | < 4 hours | < 1 hour | Customer-facing apps |
| Medium | < 24 hours | < 24 hours | Internal tools |
| Low | < 72 hours | < 72 hours | Logs, archives |

## Disaster Scenarios

| Scenario | Response |
|----------|----------|
| Server failure | Failover to standby / restore from backup |
| Data corruption | Restore database from latest backup |
| Security breach | Isolate server, restore from clean backup |
| Region outage | DNS switch to DR region |
| Accidental deletion | Restore from backup (hourly/daily) |

## Procedure

### 1. Detection & Declaration

```bash
# Failed health checks
curl -f https://example.com/health || echo "SITE DOWN"

# Monitoring alert
# On-call engineer:

# 1. Confirm outage
# 2. Declare DR event in #incidents channel
# 3. Notify DR team
```

### 2. Assess Impact

- Which services are affected?
- Estimated RTO remaining?
- Is this a partial or full disaster?
- Can it be resolved without full DR?

### 3. Activate DR Plan

#### Server Failure

```bash
# Option A: Spare server
# Update DNS to standby IP
# Mount latest backup volumes

# Option B: Restore from backup
# Provision new server
# Restore latest backup
```

#### Database Failure

```bash
# Restore from S3 backup
aws s3 cp s3://backups/db/latest.sql .
mysql -u root -p < latest.sql

# Or promote replica
mysql -e "STOP SLAVE;"
mysql -e "RESET SLAVE ALL;"
# Update app config to point to promoted replica
```

#### Full Region Outage (Cloud)

```bash
# Deploy to secondary region
terraform apply -var-file=dr.tfvars

# Update DNS to DR region
# Verify failover
```

### 4. Failover Verification

```bash
# Check service health
curl -I https://example.com

# Check database
php -r "new PDO('mysql:host=...;dbname=...', 'user', 'pass'); echo 'OK\n';"

# Check monitoring
```

### 5. Restoration

Once primary is restored:

```bash
# Sync data back
rsync -avz dr-server:/data /data

# Switch DNS back to primary
# Verify primary is healthy
# Decommission DR resources
```

## DR Testing Schedule

| Test Type | Frequency | Scope |
|-----------|-----------|-------|
| Database restore | Monthly | Single DB restore |
| Server rebuild | Quarterly | Full server provisioning |
| DNS failover | Bi-annually | Full DR drill |
| Region failover | Annually | Cross-region DR |

### DR Drill Checklist

- [ ] DR team notified and assembled
- [ ] DR plan document accessible
- [ ] Backups verified restorable
- [ ] DNS updated to DR site
- [ ] All services verified on DR site
- [ ] Monitoring working on DR site
- [ ] Failback completed
- [ ] Post-drill review scheduled

## DR Kit Location

| Item | Location |
|------|----------|
| Backups | S3 (encrypted, cross-region) |
| Terraform state | S3 backend |
| Docker images | Registry (multi-region) |
| Configuration | Vault / encrypted repo |
| DR runbook | This document |
| Access credentials | Password manager |

## Recovery Scripts

Store automation scripts in `scripts/dr/`:

```bash
scripts/dr/
├── restore-db.sh        # Database restore
├── failover-dns.sh      # DNS update
├── provision-server.sh  # Terraform apply
└── verify-health.sh     # Health checks
```

## Verification

- [ ] RTO/RPO defined for each service tier
- [ ] Backups stored off-site / cross-region
- [ ] DR plan tested within last 6 months
- [ ] Recovery scripts work
- [ ] DR contact list current
- [ ] DR documentation reviewed annually
