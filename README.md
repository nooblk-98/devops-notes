<div align="center">
  <img src="docs/img/logo.webp" alt="DevOps Docs" width="96" />

  # DevOps Reference Documentation

  A global knowledge base for DevOps engineers — operations, runbooks, and practical guides.

  [![MkDocs](https://img.shields.io/badge/MkDocs-1.6-06c6a6?style=flat-square)](https://www.mkdocs.org/)
  [![License](https://img.shields.io/badge/License-Proprietary-red?style=flat-square)](LICENSE)

  [Overview](#overview) • [Quick Start](#quick-start) • [Project Structure](#project-structure) • [Deployment](#deployment)

</div>

## Overview

A comprehensive open knowledge base for DevOps engineers worldwide. Contains runbooks, operations documentation, and practical guides covering the full DevOps lifecycle — from server provisioning and CI/CD to monitoring, security, and incident response.

The site is built with [MkDocs](https://www.mkdocs.org/), uses the [ReadTheDocs](https://github.com/cjsheets/mkdocs-rtd-dropdown) theme, and is published on [Cloudflare Pages](https://pages.cloudflare.com/). Every page shows its last revision date, code blocks have one-click copy, and images open in a lightbox.

### What you'll find here

All documentation is organized into unified categories:

| Category | Topics |
|----------|--------|
| Server Setup & Provisioning | Provisioning, Patch Management, Migration, Certbot, Docker, CloudPanel, aaPanel, Swap, File Permissions |
| Web Server & Performance | Nginx tuning, PHP tuning, Cloudflare Cache, Log management, SSL, DNS |
| Database | MySQL/MariaDB/PostgreSQL, Migrations, Redis, Memcached |
| Monitoring & Benchmarking | Prometheus/Grafana, Netdata, Server benchmarking |
| Docker | Maintenance, Compose, Staging environments |
| DevOps Tools & Automation | Git, CI/CD, WP-CLI, SSH, Cron, Ansible |
| Security | Hardening, Firewall, Malware, WAF, Incident Postmortem |
| Frontend & Deployment | WordPress, Laravel, Python/WSGI, Next.js, Node.js, Static sites |
| Backup & Disaster Recovery | Backup procedures, DR planning |


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
│   ├── sops/                # Operations documentation (merged into categories)
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
