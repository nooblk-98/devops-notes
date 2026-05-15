<div align="center">
  <img src="docs/img/logo.webp" alt="DevOps Docs" width="96" />

  # DevOps Reference Documentation

  A global reference for DevOps engineers — SOPs, runbooks, and practical guides.

  [![MkDocs](https://img.shields.io/badge/MkDocs-1.6-06c6a6?style=flat-square)](https://www.mkdocs.org/)
  [![License](https://img.shields.io/badge/License-Proprietary-red?style=flat-square)](LICENSE)

  [Overview](#overview) • [Quick Start](#quick-start) • [Project Structure](#project-structure) • [Deployment](#deployment)

</div>

## Overview

A comprehensive open reference for DevOps engineers worldwide. Contains runbooks, standard operating procedures, and practical guides covering the full DevOps lifecycle — from server provisioning and CI/CD to monitoring, security, and incident response.

The site is built with [MkDocs](https://www.mkdocs.org/), uses the [ReadTheDocs](https://github.com/cjsheets/mkdocs-rtd-dropdown) theme, and is published on [Cloudflare Pages](https://pages.cloudflare.com/). Every page shows its last revision date, code blocks have one-click copy, and images open in a lightbox.

### What you'll find here

**Standard Operating Procedures** — Formal, versioned procedures for recurring tasks. Each SOP follows a consistent template: purpose, scope, prerequisites, step-by-step instructions, rollback steps, and verification checks.

| Area | SOPs |
|------|------|
| Infrastructure | Backup & Recovery, Monitoring, DNS, Patch Management, Disaster Recovery |
| CI/CD & Development | Git Workflow, CI/CD Pipeline, CI/CD Multi-Environment, Release Management |
| Application Deployment | WordPress, Domain Change, Laravel, Python/WSGI |
| Operations | Server Migration, Incident Postmortem |

**Guides** — Hands-on guides organized by category:

| Category | Topics |
|----------|--------|
| Server Setup & Provisioning | Provisioning, Certbot, Docker, CloudPanel, aaPanel, Swap, File Permissions |
| Web Server & Performance | Nginx tuning, PHP tuning, Cloudflare Cache, Log management, SSL |
| Database | MySQL/MariaDB/PostgreSQL, Migrations, Redis, Memcached |
| Monitoring & Benchmarking | Prometheus/Grafana, Netdata, Server benchmarking |
| Docker | Maintenance, Compose, Staging environments |
| DevOps Tools & Automation | WP-CLI, SSH keys, Tunneling, Cron, Ansible |
| Security | Hardening, Firewall, Malware removal, WAF/ModSecurity, AWS WAF |
| Frontend & Deployment | Next.js + WordPress, Node.js/PM2, Static sites |
| General | Onboarding, Troubleshooting |

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
