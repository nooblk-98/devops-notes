<div align="center">
  <img src="docs/img/logo.webp" alt="DevOps Docs" width="96" />

  # DevOps SOP Documentation

  Standard Operating Procedures and guides for DevOps operations.

  [![MkDocs](https://img.shields.io/badge/MkDocs-1.6-06c6a6?style=flat-square)](https://www.mkdocs.org/)
  [![License](https://img.shields.io/badge/License-Proprietary-red?style=flat-square)](LICENSE)

  [Overview](#overview) • [Quick Start](#quick-start) • [Project Structure](#project-structure) • [Deployment](#deployment)

</div>

## Overview

This repository contains runbooks, standard operating procedures, and practical guides for day-to-day DevOps operations — from server provisioning and WordPress deployment to incident response and disaster recovery.

The site is built with [MkDocs](https://www.mkdocs.org/), uses the [ReadTheDocs](https://github.com/cjsheets/mkdocs-rtd-dropdown) theme, and is published on [Cloudflare Pages](https://pages.cloudflare.com/). Every page shows its last revision date, code blocks have one-click copy, and images open in a lightbox.

### What you'll find here

**Standard Operating Procedures** — Formal, versioned procedures for recurring tasks. Each SOP follows a consistent template: purpose, scope, prerequisites, step-by-step instructions, rollback steps, and verification checks.

| Area | SOPs |
|------|------|
| Infrastructure | Backup & Recovery, Monitoring, DNS, Patch Management, Disaster Recovery |
| Development | Git Workflow, CI/CD Pipeline, CI/CD Multi-Environment, Release Management |
| Application Deployment | WordPress, Domain Change, Laravel, Python/WSGI |
| Incidents | Incident Postmortem |
| Migration | Server Migration |

**Guides** — Hands-on guides covering everything from onboarding and tool installation to advanced configurations:

- **Setup**: Onboarding, Certbot, Docker, CloudPanel, aaPanel, Server Provisioning
- **Web Server**: Nginx tuning, PHP tuning, Log management, SSL troubleshooting
- **Database**: MySQL/MariaDB/PostgreSQL management, Redis, Memcached, Migrations
- **Monitoring**: Prometheus/Grafana, Netdata, Server benchmarking
- **Docker**: Maintenance, Compose, Staging environments
- **DevOps Tools**: WP-CLI, SSH keys, Tunneling, Cron, Ansible
- **Frontend/Backend**: Next.js + WordPress headless, Static sites, Node.js/PM2
- **Security**: Firewall rules, Hardening, Malware removal, WAF/ModSecurity, AWS WAF

**HR** — Internal references: DevOps KPI scoring, bonus and increment guidelines for FY 2026/27.

## Quick Start

### Prerequisites
- Python 3.12+
- pip

### Run locally

```bash
# Install everything you need
pip install mkdocs mkdocs-rtd-dropdown mkdocs-print-site-plugin \
  mkdocs-git-revision-date-localized-plugin mkdocs-redirects \
  mkdocs-minify-plugin mkdocs-glightbox mkdocs-section-index \
  mkdocs-table-reader-plugin

# Start editing — hot-reloads on file changes
mkdocs serve
```

Open `http://localhost:8000` in your browser.

### Build the static site

```bash
mkdocs build --strict
```

Output goes to the `site/` directory, ready for deployment.

## Project Structure

```
.
├── docs/                    # Markdown source files
│   ├── index.md             # Home page
│   ├── sops/                # Standard Operating Procedures
│   ├── guides/              # Step-by-step guides
│   ├── hr/                  # HR documentation
│   ├── img/                 # Images and assets
│   ├── css/                 # Custom styles (task lists, copy button)
│   └── js/                  # Custom JavaScript (code copy button)
├── logo/                    # Project logo
├── .github/workflows/       # GitHub Actions CI/CD
├── mkdocs.yml               # MkDocs configuration
└── wrangler.toml             # Cloudflare Pages configuration
```

## Plugins

| Plugin | What it adds |
|--------|-------------|
| [git-revision-date-localized](https://github.com/timvink/mkdocs-git-revision-date-localized) | Shows when each page was last updated and created |
| [redirects](https://github.com/datarobot/mkdocs-redirects) | Handles URL changes when pages are moved |
| [glightbox](https://github.com/blueswen/mkdocs-glightbox) | Opens screenshots and diagrams in a lightbox |
| [section-index](https://github.com/oprypin/mkdocs-section-index) | Makes nav section headings clickable |
| [table-reader](https://github.com/neilisaac/mkdocs-table-reader) | Reads CSV/JSON files as Markdown tables |
| [minify](https://github.com/byrnereese/mkdocs-minify-plugin) | Minifies HTML/JS/CSS for faster loading |
| [print-site](https://github.com/timvink/mkdocs-print-site-plugin) | Generates a printable version of the entire site |

## Deployment

Pushes to `main` trigger a GitHub Actions workflow that builds the site and deploys it to Cloudflare Pages.

## Technology

- [MkDocs](https://www.mkdocs.org/) — Static site generator
- [mkdocs-rtd-dropdown](https://github.com/cjsheets/mkdocs-rtd-dropdown) — ReadTheDocs theme with dropdown navigation
- [Cloudflare Pages](https://pages.cloudflare.com/) — Hosting and CDN
- [GitHub Actions](https://github.com/features/actions) — CI/CD automation
