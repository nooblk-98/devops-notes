---
tags:
  - backup
  - recovery
  - disaster-recovery
---

# Backup & Recovery SOP

**Version:** 1.0  
**Last Updated:** 2024-01-01  
**Owner:** DevOps Team

## Purpose

Ensure critical data is regularly backed up and can be restored in a timely manner.

## Scope

Databases, configuration files, persistent volumes, and infrastructure state.

## Backup Schedule

| Asset | Frequency | Retention | Method |
|-------|-----------|-----------|--------|
| PostgreSQL | Daily | 30 days | `pg_dump` → S3 |
| MongoDB | Daily | 14 days | `mongodump` → S3 |
| Application files | Daily | 7 days | `tar` → S3 |
| Terraform state | On change | 90 days | S3 versioning |

## Procedure

### Database Backup

```bash
# PostgreSQL backup
pg_dump -h <host> -U <user> -d <database> -Fc > backup-$(date +%Y%m%d).dump

# Upload to S3
aws s3 cp backup-$(date +%Y%m%d).dump s3://backups/postgres/

# Verify backup exists
aws s3 ls s3://backups/postgres/
```

## Recovery Procedure

### Database Restore

```bash
# Download backup from S3
aws s3 cp s3://backups/postgres/backup-20240101.dump .

# Restore PostgreSQL
pg_restore -h <host> -U <user> -d <database> -c backup-20240101.dump

# Verify data
psql -h <host> -U <user> -d <database> -c "SELECT count(*) FROM users;"
```

## Recovery Testing

!!! important "Test Schedule"
    Full recovery drills are performed quarterly. Database restores are tested monthly.

| Test | Frequency | Responsible |
|------|-----------|-------------|
| Database restore | Monthly | DB Admin |
| Full file restore | Quarterly | DevOps |
| DR failover | Bi-annually | DevOps + Infra |

## Verification

- [ ] Backup completed successfully
- [ ] Backup file size is reasonable
- [ ] Restore tested within SLA window
- [ ] Monitoring of backup jobs is active
