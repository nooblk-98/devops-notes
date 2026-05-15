---
tags:
  - cicd
  - github-actions
  - automation
---

# CI/CD Pipeline

**Version:** 1.0  
**Owner:** DevOps Team

## Purpose

Standardize CI/CD pipeline setup for automated testing, building, and deployment.

## Pipeline Stages

```
Code Push → Lint → Test → Build → Deploy Staging → Deploy Production
```

## GitHub Actions Workflow Templates

### PHP / WordPress Lint & Deploy

```yaml title=".github/workflows/deploy-wordpress.yml"
name: Deploy WordPress

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Deploy via rsync
        run: |
          rsync -avz --delete --exclude='.git' \
            -e "ssh -o StrictHostKeyChecking=no" \
            ./ ${{ secrets.DEPLOY_USER }}@${{ secrets.DEPLOY_HOST }}:/var/www/html
```

### Docker Build & Deploy

```yaml title=".github/workflows/docker-deploy.yml"
name: Docker Build & Deploy

on:
  push:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Build Docker image
        run: docker build -t app:${{ github.sha }} .

      - name: Push to registry
        run: |
          docker tag app:${{ github.sha }} ${{ secrets.REGISTRY }}/app:latest
          docker push ${{ secrets.REGISTRY }}/app:latest

      - name: Deploy via SSH
        uses: appleboy/ssh-action@v1
        with:
          host: ${{ secrets.HOST }}
          username: ${{ secrets.USER }}
          key: ${{ secrets.SSH_KEY }}
          script: |
            cd /opt/app
            docker compose pull
            docker compose up -d --force-recreate
```

### Multi-Stage Pipeline

```yaml title=".github/workflows/full-pipeline.yml"
name: CI/CD Pipeline

on:
  push:
    branches: [develop, main]
  pull_request:
    branches: [main]

jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Run linter
        run: |
          # PHP
          composer lint
          # Node
          npm run lint
          # Dockerfile
          docker run --rm -v $PWD:/code hadolint/hadolint hadolint /code/Dockerfile

  test:
    needs: lint
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Run tests
        run: |
          # PHPUnit
          vendor/bin/phpunit
          # Playwright / Cypress
          npm test

  build:
    needs: test
    if: github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Build & push image
        run: |
          docker build -t app:${{ github.sha }} .
          docker push app:${{ github.sha }}
      - name: Deploy production
        run: |
          # Deploy commands
```

## Environment Separation

| Environment | Branch | Trigger | Approval |
|-------------|--------|---------|----------|
| Development | `develop` | Push | Auto |
| Staging | `release/*` | PR to main | Auto |
| Production | `main` | Push/PR merge | Manual approval |

## Required Secrets

| Secret | Purpose |
|--------|---------|
| `SSH_PRIVATE_KEY` | SSH key for deployment server |
| `DEPLOY_HOST` | Server hostname/IP |
| `DEPLOY_USER` | SSH user |
| `REGISTRY` | Container registry URL |
| `REGISTRY_USERNAME` | Registry login |
| `REGISTRY_PASSWORD` | Registry password |

## Verification

- [ ] Pipeline runs successfully on push
- [ ] Tests pass before deployment
- [ ] Deployment completes without errors
- [ ] Health check passes post-deployment
- [ ] Slack/email notification sent
