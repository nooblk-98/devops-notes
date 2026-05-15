# Next.js Frontend with WordPress Backend

## Architecture

```
User → Nginx (host) → Next.js (Node container :3000) → WordPress API (PHP-FPM :9000) → MariaDB (:3306)
```

WordPress runs headless (REST API only) — Next.js fetches content via `wp-json/wp/v2/` and renders the frontend.

## Project Structure

```
project-root/
├── frontend/                    # Next.js application
│   ├── pages/
│   ├── components/
│   ├── lib/api.js              # WordPress API client
│   ├── next.config.js
│   ├── Dockerfile
│   └── package.json
├── wordpress/                   # WordPress (headless CMS)
│   ├── wp-content/
│   ├── Dockerfile
│   └── .env
├── nginx/
│   └── app.conf                 # Nginx config
├── docker-compose.yml
└── .github/
    └── workflows/
        └── deploy.yml
```

## Docker Compose

```yaml title="docker-compose.yml"
services:
  maria-db:
    image: mariadb:10
    container_name: wp-db
    restart: unless-stopped
    env_file: .env
    volumes:
      - ./db:/var/lib/mysql
    networks:
      - app-network
    healthcheck:
      test: ["CMD", "healthcheck.sh", "--connect", "--innodb_initialized"]
      interval: 10s
      timeout: 5s
      retries: 5

  wordpress:
    image: wordpress:php8.2-fpm
    container_name: wp-app
    restart: unless-stopped
    env_file: .env
    volumes:
      - ./wordpress:/var/www/html
    ports:
      - "127.0.0.1:9000:9000"
    depends_on:
      maria-db:
        condition: service_healthy
    networks:
      - app-network

  nextjs:
    build:
      context: ./frontend
      dockerfile: Dockerfile
    container_name: next-app
    restart: unless-stopped
    ports:
      - "127.0.0.1:3000:3000"
    environment:
      - WORDPRESS_API_URL=http://wordpress:9000/wp-json/wp/v2
      - NODE_ENV=production
    depends_on:
      - wordpress
    networks:
      - app-network

networks:
  app-network:
    driver: bridge
```

## Environment File

```bash title=".env"
# Database
MYSQL_DATABASE=wordpress
MYSQL_USER=wpuser
MYSQL_PASSWORD=strong_password
MYSQL_ROOT_PASSWORD=strong_root_password

# WordPress
WORDPRESS_DB_HOST=maria-db:3306
WORDPRESS_DB_USER=wpuser
WORDPRESS_DB_PASSWORD=strong_password
WORDPRESS_DB_NAME=wordpress
WORDPRESS_CONFIG_EXTRA=define('WP_HOME','https://api.example.com'); define('WP_SITEURL','https://api.example.com');

# WordPress Headless Settings
WP_HOME=https://api.example.com
WP_SITEURL=https://api.example.com
```

## WordPress Dockerfile

```dockerfile title="wordpress/Dockerfile"
FROM wordpress:php8.2-fpm

# Enable headless mode plugin
COPY wp-content /var/www/html/wp-content

# Install WPGraphQL (optional, for GraphQL)
RUN apt update && apt install -y git unzip \
    && curl -L https://github.com/wp-graphql/wp-graphql/releases/latest/download/wp-graphql.zip -o /tmp/wp-graphql.zip \
    && unzip /tmp/wp-graphql.zip -d /var/www/html/wp-content/plugins/ \
    && rm /tmp/wp-graphql.zip
```

## Next.js Dockerfile

```dockerfile title="frontend/Dockerfile"
FROM node:18-alpine AS deps
WORKDIR /app
COPY package.json package-lock.json ./
RUN npm ci --only=production

FROM node:18-alpine AS build
WORKDIR /app
COPY --from=deps /app/node_modules ./node_modules
COPY . .
RUN npm run build

FROM node:18-alpine AS runner
WORKDIR /app
ENV NODE_ENV production

RUN addgroup --system --gid 1001 nodejs \
    && adduser --system --uid 1001 nextjs

COPY --from=build /app/public ./public
COPY --from=build --chown=nextjs:nodejs /app/.next ./.next
COPY --from=build /app/node_modules ./node_modules
COPY --from=build /app/package.json ./package.json

USER nextjs
EXPOSE 3000

CMD ["node_modules/.bin/next", "start"]
```

## WordPress Headless Setup

### Required Plugins

- **Advanced Custom Fields (ACF)** — custom fields for content
- **ACF to REST API** — expose ACF fields via REST API
- **WP GraphQL** (optional) — GraphQL endpoint
- **Custom Post Type UI** — custom post types
- **Yoast SEO** — SEO metadata via REST API

### wp-config Additions

```php
// In wp-config.php, after database settings
define('WP_HOME', 'https://api.example.com');
define('WP_SITEURL', 'https://api.example.com');

// Disable frontend
add_filter('template_include', function() {
    return false;
}, 99);
```

### Enable Pretty Permalinks

In WordPress admin: Settings → Permalinks → Post name

## Next.js Frontend Setup

### package.json

```json title="frontend/package.json"
{
  "name": "nextjs-wordpress",
  "scripts": {
    "dev": "next dev",
    "build": "next build",
    "start": "next start"
  },
  "dependencies": {
    "next": "^14",
    "react": "^18",
    "react-dom": "^18"
  }
}
```

### WordPress API Client

```js title="frontend/lib/api.js"
const API_URL = process.env.WORDPRESS_API_URL || 'http://localhost:9000/wp-json/wp/v2'

export async function getPosts() {
  const res = await fetch(`${API_URL}/posts?_embed`)
  if (!res.ok) throw new Error('Failed to fetch posts')
  return res.json()
}

export async function getPost(slug) {
  const res = await fetch(`${API_URL}/posts?slug=${slug}&_embed`)
  const posts = await res.json()
  return posts[0] || null
}

export async function getPages() {
  const res = await fetch(`${API_URL}/pages?_embed`)
  return res.json()
}
```

### Pages Setup

```jsx title="frontend/pages/index.js"
import { getPosts } from '../lib/api'

export default function Home({ posts }) {
  return (
    <div>
      <h1>Blog</h1>
      {posts.map(post => (
        <article key={post.id}>
          <h2>{post.title.rendered}</h2>
          <div dangerouslySetInnerHTML={{ __html: post.excerpt.rendered }} />
        </article>
      ))}
    </div>
  )
}

export async function getStaticProps() {
  const posts = await getPosts()
  return { props: { posts }, revalidate: 60 }
}
```

```jsx title="frontend/pages/posts/[slug].js"
import { getPosts, getPost } from '../../lib/api'

export default function Post({ post }) {
  return (
    <article>
      <h1>{post.title.rendered}</h1>
      <div dangerouslySetInnerHTML={{ __html: post.content.rendered }} />
    </article>
  )
}

export async function getStaticPaths() {
  const posts = await getPosts()
  const paths = posts.map(post => ({ params: { slug: post.slug } }))
  return { paths, fallback: 'blocking' }
}

export async function getStaticProps({ params }) {
  const post = await getPost(params.slug)
  return { props: { post }, revalidate: 60 }
}
```

### next.config.js

```js title="frontend/next.config.js"
/** @type {import('next').NextConfig} */
const nextConfig = {
  images: {
    remotePatterns: [
      { protocol: 'https', hostname: 'api.example.com' },
    ],
  },
  async rewrites() {
    return [
      { source: '/api/:path*', destination: 'http://wordpress:9000/wp-json/wp/v2/:path*' },
    ]
  },
}

module.exports = nextConfig
```

## Nginx Configuration

```nginx title="/etc/nginx/sites-available/example.com"
server {
    listen 80;
    server_name example.com api.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name example.com;

    ssl_certificate /etc/letsencrypt/live/example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem;

    location / {
        proxy_pass http://127.0.0.1:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}

# WordPress API subdomain
server {
    listen 443 ssl http2;
    server_name api.example.com;

    ssl_certificate /etc/letsencrypt/live/api.example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/api.example.com/privkey.pem;

    root /opt/nextjs-wordpress/wordpress;
    index index.php;

    location / {
        try_files $uri $uri/ /index.php?$args;
    }

    location ~ \.php$ {
        fastcgi_pass 127.0.0.1:9000;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
    }
}
```

## Deployment

```bash
# Build and start
docker compose build
docker compose up -d

# Install WordPress plugins (first run)
docker compose exec wordpress wp plugin install acf-to-rest-api --activate
docker compose exec wordpress wp plugin install advanced-custom-fields --activate

# Create admin user
docker compose exec wordpress wp user create admin admin@example.com --role=administrator --user_pass=password

# Verify both services
curl -I http://localhost:3000          # Next.js
curl http://localhost:9000/wp-json/wp/v2/posts  # WordPress API
```

## GitHub Actions

```yaml title=".github/workflows/deploy.yml"
name: Deploy Next.js + WordPress

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Deploy via SSH
        uses: appleboy/ssh-action@v1
        with:
          host: ${{ secrets.DEPLOY_HOST }}
          username: ${{ secrets.DEPLOY_USER }}
          key: ${{ secrets.SSH_PRIVATE_KEY }}
          script: |
            cd /opt/nextjs-wordpress
            git pull origin main
            docker compose build
            docker compose up -d --force-recreate
            docker image prune -f
```

## Troubleshooting

| Symptom | Cause | Fix |
|---------|-------|-----|
| Next.js can't reach WordPress API | Network isolation | Ensure both on same Docker network |
| Blank WordPress admin | Wrong WP_HOME/SITEURL | Check `.env` URL values |
| Next.js build fails | API not ready during build | Use `fallback: 'blocking'` in `getStaticPaths` |
| Images broken | Missing remotePatterns | Add domain in `next.config.js` |
| 502 on Next.js | Node container not ready | `docker compose restart nextjs` |

## Verification

- [ ] WordPress API accessible at `api.example.com/wp-json/wp/v2/posts`
- [ ] Next.js frontend loads at `example.com`
- [ ] ISR revalidation working (content updates within 60s)
- [ ] Docker containers healthy
- [ ] SSL certificates valid for both domains
- [ ] GitHub Actions deploy succeeds
