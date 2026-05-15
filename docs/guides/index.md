# All Documentation

## Server Setup & Provisioning
- [Server Provisioning](server-provisioning.md) — Initial server setup and hardening
- [Patch Management](../sops/patch-management.md) — Scheduled updates, kernel upgrades, reboot cycles
- [Server Migration](../sops/migration.md) — Full server and site migration process
- [Install Certbot & SSL](install-certbot.md) — Install Certbot and obtain SSL certificates
- [Install Docker & Compose](install-docker.md) — Install Docker Engine and Compose plugin
- [Install CloudPanel](install-cloudpanel.md) — Install CloudPanel control panel
- [Install aaPanel](install-aapanel.md) — Install aaPanel control panel
- [Swap Configuration](swap-configuration.md) — When and how to add swap
- [File Permissions Cheat Sheet](file-permissions.md) — Standard ownership for web apps

## Web Server & Performance
- [Nginx Performance Tuning](nginx-performance.md) — Worker processes, buffers, gzip, caching
- [PHP Tuning](php-tuning.md) — PHP-FPM, OPcache, and performance tuning
- [Cloudflare Cache Tuning](cloudflare-cache.md) — Cache rules, purge strategies, ARGO
- [Log Management](log-management.md) — Logrotate, centralized logging with Loki/ELK
- [SSL Troubleshooting](ssl-troubleshooting.md) — Common cert errors, chain issues, mixed content
- [DNS Management](../sops/dns-management.md) — DNS records, Cloudflare, DNSSEC

## Database
- [Database Management](database-management.md) — MySQL, MariaDB, PostgreSQL management and tuning
- [Database Migration](database-migration.md) — MySQL version upgrade, charset conversion
- [Redis & Memcached](redis-memcached.md) — Install and configure as object cache

## Monitoring & Benchmarking
- [Monitoring](../sops/monitoring.md) — Infrastructure and application monitoring
- [Monitoring Setup](monitoring-setup.md) — Node Exporter, Prometheus, Grafana installation
- [Netdata Monitoring](netdata-monitoring.md) — Real-time server monitoring with Netdata
- [Server Benchmarking](server-benchmarking.md) — Load testing and benchmarking tools

## Docker
- [Docker Maintenance](docker-maintenance.md) — Cleanup, backup, updates, and security
- [Docker Compose Cheat Sheet](docker-compose-cheatsheet.md) — Compose commands, healthchecks, networking
- [Staging Environment Setup](staging-environment.md) — Cloning production to staging

## DevOps Tools & Automation
- [Git Workflow](../sops/git-workflow.md) — Branching, commits, code review process
- [CI/CD Pipeline](../sops/cicd-pipeline.md) — Automated testing and deployment pipelines
- [CI/CD Multi-Environment](../sops/cicd-multi-env.md) — Dev/staging/prod deployment strategy
- [Release Management](../sops/release-management.md) — Versioning, changelogs, release notes
- [WP-CLI Cheat Sheet](wp-cli-cheatsheet.md) — Common wp-cli commands reference
- [SSH Key Management](ssh-key-management.md) — Generate, deploy, rotate keys
- [SSH Tunneling](ssh-tunneling.md) — Port forwarding, jump hosts, reverse tunnels
- [Cron Job Management](cron-management.md) — Scheduling, monitoring, common crons
- [Ansible Automation](ansible-automation.md) — Server provisioning with Ansible playbooks

## Security
- [Security Practices](security-practices.md) — Hardening and security checklist
- [WordPress Firewall Rules](firewall-wordpress.md) — Standard firewall rules for WordPress
- [Malware Removal](malware-removal.md) — Steps to clean an infected site
- [.htaccess Hardening](htaccess-hardening.md) — IP blocking, hotlink protection
- [WAF / ModSecurity](modsecurity-waf.md) — Web application firewall rules
- [AWS WAF for WordPress](aws-waf-wordpress.md) — AWS WAF with managed rules and rate limiting
- [Incident Postmortem](../sops/incident-postmortem.md) — Postmortem template and process

## Frontend & Deployment
- [WordPress Deployment](../sops/wordpress-deployment.md) — WordPress with Docker, Nginx, and MariaDB
- [WordPress Domain Change](../sops/wordpress-change-url.md) — Change WordPress site URL
- [Laravel Deployment](../sops/laravel-deployment.md) — Laravel with Docker, Nginx, and MariaDB
- [Python/WSGI Deployment](../sops/python-wsgi-deployment.md) — Django/Flask with Gunicorn and Docker
- [Next.js + WordPress](nextjs-wordpress.md) — Next.js frontend with headless WordPress backend
- [Node.js/PM2 Deployment](nodejs-pm2-deployment.md) — Node.js apps with PM2 and Nginx
- [Static Site Deployment](static-site-deployment.md) — Hugo/11ty build and deploy

## Backup & Disaster Recovery
- [Backup & Recovery](../sops/backup.md) — Data protection and disaster recovery
- [Disaster Recovery](../sops/disaster-recovery.md) — DR plan, RTO/RPO, failover drills

## General
- [Onboarding Guide](onboarding.md) — Setting up a new team member's environment
- [Troubleshooting Guide](troubleshooting.md) — Common issues and resolutions
