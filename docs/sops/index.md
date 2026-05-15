# Operations

Reference documentation for common DevOps operations and procedures.

## Available Operations

### Infrastructure
- [Backup & Recovery](backup.md) — Data protection and disaster recovery
- [Monitoring](monitoring.md) — Infrastructure and application monitoring
- [DNS Management](dns-management.md) — DNS records, Cloudflare, DNSSEC
- [Patch Management](patch-management.md) — Scheduled updates, kernel upgrades, reboot cycles
- [Disaster Recovery](disaster-recovery.md) — DR plan, RTO/RPO, failover drills

### CI/CD & Development
- [Git Workflow](git-workflow.md) — Branching, commits, code review process
- [CI/CD Pipeline](cicd-pipeline.md) — Automated testing and deployment pipelines
- [CI/CD Multi-Environment](cicd-multi-env.md) — Dev/staging/prod deployment strategy
- [Release Management](release-management.md) — Versioning, changelogs, release notes

### Application Deployment
- [WordPress Deployment](wordpress-deployment.md) — WordPress with Docker, Nginx, and MariaDB
- [WordPress Domain Change](wordpress-change-url.md) — Change WordPress site URL
- [Laravel Deployment](laravel-deployment.md) — Laravel with Docker, Nginx, and MariaDB
- [Python/WSGI Deployment](python-wsgi-deployment.md) — Django/Flask with Gunicorn and Docker

### Procedures
- [Server Migration](migration.md) — Full server and site migration process
- [Incident Postmortem](incident-postmortem.md) — Postmortem template and process

## Document Structure

Each document follows this structure:

1. **Purpose** — What this SOP covers
2. **Scope** — When to use this procedure
3. **Prerequisites** — Required access, tools, and permissions
4. **Procedure** — Step-by-step instructions
5. **Rollback** — How to revert if something goes wrong
6. **Verification** — How to confirm success
