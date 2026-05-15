# Standard Operating Procedures

This section contains all Standard Operating Procedures (SOPs) for DevOps operations.

## Available SOPs

### Infrastructure
- [Backup & Recovery SOP](backup.md) — Data protection and disaster recovery
- [Monitoring SOP](monitoring.md) — Infrastructure and application monitoring
- [DNS Management SOP](dns-management.md) — DNS records, Cloudflare, DNSSEC
- [Patch Management SOP](patch-management.md) — Scheduled updates, kernel upgrades, reboot cycles
- [Disaster Recovery SOP](disaster-recovery.md) — DR plan, RTO/RPO, failover drills

### Development
- [Git Workflow SOP](git-workflow.md) — Branching, commits, code review process
- [CI/CD Pipeline SOP](cicd-pipeline.md) — Automated testing and deployment pipelines
- [CI/CD Multi-Environment SOP](cicd-multi-env.md) — Dev/staging/prod deployment strategy
- [Release Management SOP](release-management.md) — Versioning, changelogs, release notes

### Application Deployment
- [WordPress Deployment SOP](wordpress-deployment.md) — WordPress with Docker, Nginx, and MariaDB
- [WordPress Domain Change SOP](wordpress-change-url.md) — Change WordPress site URL
- [Laravel Deployment SOP](laravel-deployment.md) — Laravel with Docker, Nginx, and MariaDB
- [Python/WSGI Deployment SOP](python-wsgi-deployment.md) — Django/Flask with Gunicorn and Docker

### Incidents
- [Incident Postmortem SOP](incident-postmortem.md) — Postmortem template and process

### Migration
- [Server Migration SOP](migration.md) — Full server and site migration process

## SOP Template

Each SOP follows this structure:

1. **Purpose** — What this SOP covers
2. **Scope** — When to use this procedure
3. **Prerequisites** — Required access, tools, and permissions
4. **Procedure** — Step-by-step instructions
5. **Rollback** — How to revert if something goes wrong
6. **Verification** — How to confirm success
