<div align="center">
  <img src="docs/img/logo.webp" alt="" width="96" />

  # DevOps Reference

  *A comprehensive knowledge base for DevOps engineers — server provisioning, CI/CD, monitoring, security, and everything in between.*

  [![MkDocs](https://img.shields.io/badge/MkDocs-1.6-06c6a6?style=flat-square&logo=markdown)](https://www.mkdocs.org/)
  [![Build](https://img.shields.io/github/actions/workflow/status/nooblk-98/devops-notes/deploy.yml?style=flat-square&label=Deploy&logo=github)](https://github.com/nooblk-98/devops-notes/actions)
  [![License](https://img.shields.io/badge/License-Proprietary-red?style=flat-square)](LICENSE)
  [![Pages](https://img.shields.io/badge/Cloudflare-Pages-F38020?style=flat-square&logo=cloudflare)](https://devops-notes-b2c.pages.dev)

  [Overview](#overview) • [Categories](#categories) • [Quick start](#quick-start) • [Features](#features) • [Deployment](#deployment)

</div>

## Overview

This repository is a curated collection of practical DevOps knowledge covering the full infrastructure lifecycle. It contains runbooks, configuration guides, and step-by-step procedures for everything from bare-metal server provisioning to CI/CD pipeline management, monitoring stacks, security hardening, and disaster recovery.

The documentation is organized by **topic** rather than document type, so you can find relevant information whether you need a formal procedure or a quick reference.

## Categories

| Area | Topics covered |
|------|---------------|
| **AI** | Auto release with AI-generated notes, AI coding skills |
| **CI/CD Pipelines** | Git workflow, CI/CD pipelines (multi-environment), release management, Cloudflare deploy |
| **Installation Guides** | Certbot/SSL, Docker, CloudPanel, aaPanel, Jenkins, Node.js |
| **Server Setup & Provisioning** | Server hardening, patch management, migrations, swap, file permissions |
| **Web Server & Performance** | Nginx tuning, PHP-FPM, Cloudflare cache, log management, SSL, DNS |
| **Database** | MySQL / MariaDB / PostgreSQL, migrations, Redis, Memcached |
| **Monitoring & Benchmarking** | Prometheus + Grafana, Netdata, server benchmarking |
| **Docker** | Maintenance, Compose patterns, staging environments |
| **DevOps Tools** | WP-CLI, SSH keys, tunneling, cron, Ansible |
| **Security** | Hardening, firewall rules, malware removal, ModSecurity, AWS WAF, postmortems |
| **Frontend & Deployment** | WordPress, Laravel, Python/WSGI, Next.js + headless WordPress, Node.js/PM2, static sites |
| **Backup & DR** | Backup schedules, retention, DR planning with RTO/RPO |

## Quick start

### Prerequisites

- Python 3.12+
- pip

### Serve locally

```bash
pip install mkdocs mkdocs-rtd-dropdown mkdocs-print-site-plugin \
  mkdocs-git-revision-date-localized-plugin mkdocs-redirects \
  mkdocs-minify-plugin mkdocs-glightbox mkdocs-section-index \
  mkdocs-table-reader-plugin

mkdocs serve
```

Open [http://localhost:8000](http://localhost:8000) in your browser. The site auto-reloads on file changes.

### Build the static site

```bash
mkdocs build --strict
```

Output goes to the `site/` directory, ready for deployment.

## Features

- **Topic-based navigation** — All content is organized by subject, not document type. No more hunting across sections.
- **Last-updated dates** — Every page shows when it was last revised (and when it was created).
- **Image lightbox** — Screenshots and diagrams open in a lightbox for closer inspection.
- **Print-ready** — A single-page printable version of the entire documentation is available.
- **One-click copy** — Code blocks include a copy button.
- **Full-text search** — Built-in search across all documentation.
- **Strict builds** — `mkdocs build --strict` catches broken links and warnings before they reach production.

## Project structure

```
.
├── docs/
│   ├── index.md              # Home
│   ├── guides/               # Step-by-step guides
│   └── sops/                 # Operations / runbooks
├── logo/                     # Assets
├── mkdocs.yml                # Site configuration
├── wrangler.toml             # Cloudflare Pages config
└── .github/workflows/        # CI/CD pipelines
```

## Technology

| Layer | |
|-------|---|
| Site generator | [MkDocs](https://www.mkdocs.org/) 1.6 |
| Theme | [ReadTheDocs](https://github.com/cjsheets/mkdocs-rtd-dropdown) with dropdown navigation |
| Plugins | git-revision-date-localized, redirects, glightbox, section-index, table-reader, minify, print-site, search |
| Hosting | [Cloudflare Pages](https://pages.cloudflare.com/) |
| CI/CD | [GitHub Actions](https://github.com/features/actions) |

## Deployment

Pushes to the `main` branch trigger a GitHub Actions workflow that builds the site and deploys it to Cloudflare Pages.

> [!NOTE]
> The live site is available at [devops-notes-b2c.pages.dev](https://devops-notes-b2c.pages.dev).  
> The site is not indexed by search engines.
