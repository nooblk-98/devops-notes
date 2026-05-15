# CI/CD for Multiple Environments

**Version:** 1.0  
**Owner:** DevOps Team

## Architecture

```
Feature branch → Develop → Staging → Production
     │              │          │          │
   GitHub PR      Auto      Auto     Manual +
     build       deploy    deploy    auto deploy
```

## Environment Matrix

| Environment | Branch | Trigger | Approval | URL |
|-------------|--------|---------|----------|-----|
| Development | `develop` | Push | Auto | dev.example.com |
| Staging | `release/*` | PR to main | Auto | staging.example.com |
| Production | `main` | Push | Manual | example.com |

## Workflow Structure

```
.github/workflows/
├── ci.yml              # Build & test (all branches)
├── deploy-dev.yml      # Auto-deploy develop
├── deploy-staging.yml  # Auto-deploy release branches
└── deploy-prod.yml     # Manual deploy main
```

## CI (Build & Test)

```yaml title=".github/workflows/ci.yml"
name: CI

on:
  push:
    branches: [develop, main, release/**]
  pull_request:
    branches: [main]

jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Lint
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
    services:
      mysql:
        image: mariadb:10
        env:
          MYSQL_ALLOW_EMPTY_PASSWORD: yes
          MYSQL_DATABASE: test
        ports:
          - 3306:3306
    steps:
      - uses: actions/checkout@v4
      - name: Run tests
        run: |
          cp .env.testing .env
          composer install
          php artisan migrate --env=testing
          php vendor/bin/phpunit

  build:
    needs: test
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Build Docker image
        run: |
          docker build -t app:${{ github.sha }} .
      - name: Push to registry
        run: |
          echo ${{ secrets.REGISTRY_PASSWORD }} | docker login ${{ secrets.REGISTRY }} -u ${{ secrets.REGISTRY_USER }} --password-stdin
          docker tag app:${{ github.sha }} ${{ secrets.REGISTRY }}/app:latest
          docker tag app:${{ github.sha }} ${{ secrets.REGISTRY }}/app:${{ github.sha }}
          docker push --all-tags ${{ secrets.REGISTRY }}/app
```

## Deploy to Development

```yaml title=".github/workflows/deploy-dev.yml"
name: Deploy to Development

on:
  push:
    branches: [develop]

jobs:
  deploy:
    runs-on: ubuntu-latest
    environment: development
    steps:
      - uses: actions/checkout@v4

      - name: Deploy
        uses: appleboy/ssh-action@v1
        with:
          host: ${{ secrets.DEV_HOST }}
          username: ${{ secrets.DEV_USER }}
          key: ${{ secrets.SSH_KEY }}
          script: |
            cd /opt/app
            git pull origin develop
            docker compose -f docker-compose.yml -f docker-compose.dev.yml up -d --build
            docker compose exec -T app php artisan migrate --force
            echo "Deployed to development"
```

## Deploy to Staging

```yaml title=".github/workflows/deploy-staging.yml"
name: Deploy to Staging

on:
  pull_request:
    types: [opened, synchronize, reopened]
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    environment: staging
    if: startsWith(github.head_ref, 'release/')
    steps:
      - uses: actions/checkout@v4

      - name: Deploy
        uses: appleboy/ssh-action@v1
        with:
          host: ${{ secrets.STAGING_HOST }}
          username: ${{ secrets.STAGING_USER }}
          key: ${{ secrets.SSH_KEY }}
          script: |
            cd /opt/app
            git fetch origin
            git checkout ${{ github.head_ref }}
            docker compose -f docker-compose.yml -f docker-compose.staging.yml up -d --build
            docker compose exec -T app php artisan migrate --force
            echo "Staging: ${{ github.event.pull_request.html_url }}"

      - name: Comment PR
        uses: actions/github-script@v7
        with:
          script: |
            github.rest.issues.createComment({
              issue_number: context.issue.number,
              owner: context.repo.owner,
              repo: context.repo.repo,
              body: `🚀 Deployed to staging: https://staging.example.com`
            })
```

## Deploy to Production

```yaml title=".github/workflows/deploy-prod.yml"
name: Deploy to Production

on:
  push:
    branches: [main]

jobs:
  approve:
    runs-on: ubuntu-latest
    environment: production
    steps:
      - run: echo "Waiting for manual approval"

  deploy:
    needs: approve
    runs-on: ubuntu-latest
    environment: production
    steps:
      - uses: actions/checkout@v4

      - name: Deploy
        uses: appleboy/ssh-action@v1
        with:
          host: ${{ secrets.PROD_HOST }}
          username: ${{ secrets.PROD_USER }}
          key: ${{ secrets.SSH_KEY }}
          script: |
            cd /opt/app
            git pull origin main
            docker compose pull app
            docker compose up -d --force-recreate app
            docker compose exec -T app php artisan migrate --force
            docker image prune -f

      - name: Health check
        run: |
          sleep 10
          curl -f https://example.com/health

      - name: Notify
        run: |
          curl -X POST -H "Content-type: application/json" \
            --data '{"text":"🚀 Production deploy completed: ${{ github.sha }}"}' \
            ${{ secrets.SLACK_WEBHOOK }}
```

## Environment Files Strategy

```
.env          # Base (committed, example values)
.env.dev      # Development overrides
.env.staging  # Staging overrides
.env.prod     # Production (secrets, NOT committed)
```

### .env.example

```bash
APP_NAME=App
APP_ENV=local
APP_DEBUG=true
APP_URL=http://localhost

DB_HOST=db
DB_DATABASE=app
```

### Load by Environment

```bash
cp .env.example .env
cp .env.prod .env  # On production server
```

## Docker Compose Per Environment

```yaml title="docker-compose.override.yml"
# Auto-loaded for development
services:
  app:
    volumes:
      - ./app:/var/www/html
    ports:
      - "5173:5173" # Vite
```

```yaml title="docker-compose.staging.yml"
services:
  app:
    build:
      args:
        - APP_ENV=staging
    ports:
      - "127.0.0.1:9000:9000"
```

```yaml title="docker-compose.prod.yml"
services:
  app:
    build:
      args:
        - APP_ENV=production
    ports:
      - "127.0.0.1:9000:9000"
    restart: always
```

## Secrets Per Environment

Store in GitHub → Settings → Environments:

| Environment | Secret | Example |
|-------------|--------|---------|
| development | `DEV_HOST` | `dev.example.com` |
| staging | `STAGING_HOST` | `staging.example.com` |
| production | `PROD_HOST` | `prod.example.com` |
| production | `SLACK_WEBHOOK` | `https://hooks.slack.com/...` |

## Rollback

```yaml
# In deploy-prod.yml, add rollback step:
- name: Rollback (if needed)
  if: failure()
  run: |
    ssh user@host "cd /opt/app && docker compose up -d app:previous-tag"
```

## Verification

- [ ] CI runs lint, test, build on every push
- [ ] Develop auto-deploys to dev environment
- [ ] Release branches auto-deploy to staging
- [ ] Production requires manual approval
- [ ] Each environment has its own secrets
- [ ] Environment-specific docker-compose files
- [ ] Rollback strategy documented
- [ ] Notifications configured for each environment
