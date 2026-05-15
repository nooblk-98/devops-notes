# Static Site Deployment (Hugo / 11ty)

## Architecture

```
User → Nginx (host) → Static files on disk (/var/www/html)
```

## Hugo

### Project Setup

```bash
# Install Hugo
apt install -y hugo
# OR: download from https://github.com/gohugoio/hugo/releases

# Create new site
hugo new site my-site --format yaml
cd my-site

# Add theme (git submodule)
git init
git submodule add https://github.com/theNewDynamic/gohugo-theme-ananke themes/ananke

# Create content
hugo new posts/first-post.md
```

### Build

```bash
# Build to ./public
hugo

# Build with base URL
hugo --baseURL "https://example.com"

# Development server
hugo server -D

# Minify output
hugo --minify
```

### Dockerfile

```dockerfile title="Dockerfile"
FROM hugomods/hugo:latest AS builder
WORKDIR /src
COPY . .
RUN hugo --minify --baseURL "https://example.com"

FROM nginx:alpine
COPY --from=builder /src/public /usr/share/nginx/html
COPY nginx.conf /etc/nginx/conf.d/default.conf
```

```yaml title="docker-compose.yml"
services:
  app:
    build: .
    ports:
      - "127.0.0.1:8080:80"
```

## 11ty (Eleventy)

### Project Setup

```bash
npm init -y
npm install @11ty/eleventy --save-dev
```

```json title="package.json"
{
  "scripts": {
    "build": "eleventy",
    "serve": "eleventy --serve"
  }
}
```

### Build

```bash
npm run build
# Output in ./_site
```

### Dockerfile

```dockerfile title="Dockerfile"
FROM node:18-alpine AS builder
WORKDIR /app
COPY package.json package-lock.json ./
RUN npm ci
COPY . .
RUN npm run build

FROM nginx:alpine
COPY --from=builder /app/_site /usr/share/nginx/html
COPY nginx.conf /etc/nginx/conf.d/default.conf
```

## Nginx Config

```nginx title="nginx.conf"
server {
    listen 80;
    server_name example.com;
    root /usr/share/nginx/html;
    index index.html;

    # Gzip
    gzip on;
    gzip_types text/html text/css text/javascript application/javascript image/svg+xml;
    gzip_min_length 256;

    # Cache static assets
    location ~* \.(css|js|jpg|jpeg|png|gif|ico|webp|svg|woff2?)$ {
        expires 30d;
        add_header Cache-Control "public, immutable";
    }

    # HTML (short cache for updates)
    location ~* \.html$ {
        expires 1h;
        add_header Cache-Control "public, must-revalidate";
    }

    # Clean URLs
    location / {
        try_files $uri $uri/ $uri.html =404;
    }

    # Security headers
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header Referrer-Policy "strict-origin-when-cross-origin" always;
}
```

## Deployment

```bash
# Option 1: Direct copy to server
rsync -avz --delete ./public/ user@server:/var/www/html/

# Option 2: Docker
docker compose build
docker compose up -d

# Option 3: Cloudflare Pages (recommended)
npx wrangler pages deploy ./public --project-name=my-site
```

## GitHub Actions

### Hugo

```yaml
name: Deploy Hugo

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          submodules: true

      - uses: peaceiris/actions-hugo@v2
        with:
          hugo-version: "latest"

      - name: Build
        run: hugo --minify --baseURL "https://example.com"

      - name: Deploy to server
        uses: appleboy/scp-action@v0.1.4
        with:
          host: ${{ secrets.HOST }}
          username: ${{ secrets.USER }}
          key: ${{ secrets.SSH_KEY }}
          source: "public/*"
          target: "/var/www/html"
          strip_components: 1
```

### 11ty

```yaml
name: Deploy 11ty

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: "18"

      - name: Build
        run: |
          npm ci
          npm run build

      - name: Deploy to server
        uses: appleboy/scp-action@v0.1.4
        with:
          host: ${{ secrets.HOST }}
          username: ${{ secrets.USER }}
          key: ${{ secrets.SSH_KEY }}
          source: "_site/*"
          target: "/var/www/html"
          strip_components: 1
```

## Verification

- [ ] Site builds without errors
- [ ] All pages accessible
- [ ] Images and assets load
- [ ] Gzip compression enabled
- [ ] Cache headers set
- [ ] Security headers present
- [ ] CI/CD deploy works
