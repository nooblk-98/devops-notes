# Cloudflare Pages Deploy Pipeline

GitHub Actions workflow to build and deploy an MkDocs site to Cloudflare Pages.

## Workflow

Location: `.github/workflows/deploy.yml`

```yaml
name: Deploy to Cloudflare Pages

on:
  push:
    branches: [main]
  workflow_dispatch:

jobs:
  deploy:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      deployments: write

    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - uses: actions/setup-python@v5
        with:
          python-version: "3.12"

      - name: Install dependencies
        run: |
          pip install --no-cache-dir mkdocs mkdocs-rtd-dropdown mkdocs-print-site-plugin mkdocs-git-revision-date-localized-plugin mkdocs-redirects mkdocs-minify-plugin mkdocs-glightbox mkdocs-section-index mkdocs-table-reader-plugin

      - name: Build site
        run: mkdocs build --strict

      - name: Deploy to Cloudflare Pages
        uses: cloudflare/wrangler-action@v3
        with:
          apiToken: ${{ secrets.CLOUDFLARE_API_TOKEN }}
          accountId: ${{ secrets.CLOUDFLARE_ACCOUNT_ID }}
          command: pages deploy site --project-name=devops-notes
```

## Required Secrets

| Secret | Purpose |
|--------|---------|
| `CLOUDFLARE_API_TOKEN` | Cloudflare API token with Pages: Write permission |
| `CLOUDFLARE_ACCOUNT_ID` | Cloudflare account ID |

## How It Works

1. **Trigger:** Runs on every push to `main` or manually via `workflow_dispatch`.
2. **Checkout:** Full git history is fetched (`fetch-depth: 0`) for the git-revision-date plugin.
3. **Setup Python:** Python 3.12 is installed.
4. **Install Dependencies:** MkDocs and all required plugins are installed via pip.
5. **Build:** `mkdocs build --strict` generates the static site in the `site/` directory (strict mode fails on any warning).
6. **Deploy:** The `site/` directory is deployed to Cloudflare Pages using Wrangler.
