<div align="center">
  <img src="docs/img/logo.webp" alt="DevOps Reference" width="96" />

  # DevOps Reference Documentation

  A comprehensive knowledge base for DevOps engineers — operations runbooks, configuration guides, and deployment procedures.

  [![MkDocs](https://img.shields.io/badge/MkDocs-1.6-06c6a6?style=flat-square&logo=markdown)](https://www.mkdocs.org/)
  [![Build](https://img.shields.io/badge/Build-GitHub%20Actions-2088FF?style=flat-square&logo=githubactions)](https://github.com/features/actions)
  [![License](https://img.shields.io/badge/License-Proprietary-red?style=flat-square)](LICENSE)

  [Overview](#overview) • [Categories](#categories) • [Quick Start](#quick-start) • [Project Structure](#project-structure) • [Technology](#technology)

</div>

## Overview

This repository is a curated collection of practical DevOps knowledge covering the full infrastructure lifecycle. It includes runbooks, standard procedures, and step-by-step guides for everything from bare-metal server provisioning to Kubernetes deployment, CI/CD pipeline management, monitoring, security hardening, and disaster recovery.

The documentation is organized by topic rather than document type, making it easy to find relevant information regardless of whether you're looking for a formal procedure or a quick reference.

## Categories

| Category | Contents |
|----------|----------|
| **CI/CD Pipelines** | Git workflow, auto release, CI/CD pipelines, release management |
| **Installation Guides** | Certbot/SSL, Docker Engine, CloudPanel, aaPanel, Jenkins, Node.js |
| **Server Setup & Provisioning** | Server hardening, patch management, migrations, swap config, file permissions |
| **Web Server & Performance** | Nginx tuning, PHP-FPM optimization, Cloudflare caching, log management, SSL troubleshooting, DNS management |
| **Database** | MySQL/MariaDB/PostgreSQL administration, database migrations, Redis and Memcached setup |
| **Monitoring & Benchmarking** | Prometheus + Grafana stack, Netdata real-time monitoring, server benchmarking tools |
| **Docker** | Container maintenance, Docker Compose patterns, staging environment replication |
| **DevOps Tools & Automation** | WP-CLI, SSH key management, tunneling, cron jobs, Ansible playbooks |
| **Security** | Hardening checklists, WordPress firewall rules, malware removal, .htaccess hardening, ModSecurity WAF, AWS WAF, incident postmortems |
| **Frontend & Deployment** | WordPress (Docker/Nginx/MariaDB), Laravel, Python/WSGI, Next.js + headless WordPress, Node.js/PM2, static site deployment |
| **Backup & Disaster Recovery** | Backup schedules and retention, DR planning with RTO/RPO |

## Quick Start

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

### Build static site

```bash
mkdocs build --strict
```

Output goes to the `site/` directory.

## Project Structure

```
.
├── docs/
│   ├── index.md              # Home page
│   ├── guides/               # Step-by-step guides and references
│   └── sops/                 # Operations documentation and runbooks
├── logo/                     # Source logo assets
├── mkdocs.yml                # MkDocs configuration
├── wrangler.toml             # Cloudflare Pages configuration
└── .github/workflows/        # CI/CD pipelines
```

## Features

- **Topic-based navigation** — All documentation is organized by subject, not document type, making it easy to find what you need.
- **Last-updated dates** — Every page shows when it was last revised.
- **Image lightbox** — Screenshots and diagrams open in a lightbox for closer inspection.
- **Print-ready** — A single-page printable version is available.
- **One-click code copy** — Code blocks include a copy button.
- **Search** — Full-text search across all documentation.
- **Strict builds** — `mkdocs build --strict` catches broken links and warnings before deployment.

## Technology

| Layer | Technology |
|-------|------------|
| Site generator | [MkDocs](https://www.mkdocs.org/) 1.6 |
| Theme | [ReadTheDocs](https://github.com/cjsheets/mkdocs-rtd-dropdown) (dropdown navigation) |
| Plugins | git-revision-date-localized, redirects, glightbox, section-index, table-reader, minify, print-site, search |
| Hosting | [Cloudflare Pages](https://pages.cloudflare.com/) |
| CI/CD | [GitHub Actions](https://github.com/features/actions) |

## Deployment

Pushes to the `main` branch trigger a GitHub Actions workflow that builds the site and deploys it to Cloudflare Pages.

> The live site is available at [devops-notes-b2c.pages.dev](https://devops-notes-b2c.pages.dev).
>
> The site is not indexed by search engines (robots.txt disallows all crawling).
