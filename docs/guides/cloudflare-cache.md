# Cloudflare Cache Tuning

## Cache Configuration

### Default Cache Behavior

| Resource Type | Cache Duration | Cloudflare Default |
|--------------|----------------|-------------------|
| Static assets (CSS, JS, images) | 30 days | Standard |
| HTML | Bypass (default) | Bypass |
| API responses | Bypass | Bypass |

### Enable Cache on HTML

Using Cache Rules (recommended for WordPress):

```
Rule: example.com/wp-content/*.css
Cache Level: Cache Everything
Edge TTL: 7 days

Rule: example.com/wp-content/*.js
Cache Level: Cache Everything
Edge TTL: 7 days

Rule: example.com/wp-content/uploads/*
Cache Level: Cache Everything
Edge TTL: 30 days
```

### Page Rules (Legacy)

```yaml
# Cache static assets aggressively
example.com/wp-content/*:
  - Cache Level: Cache Everything
  - Edge Cache TTL: 7 days

# Bypass admin and login
example.com/wp-admin/*:
  - Cache Level: Bypass
  - Disable Performance: On

example.com/wp-login.php:
  - Cache Level: Bypass
  - Disable Performance: On

# Bypass cart/checkout (WooCommerce)
example.com/cart/*:
  - Cache Level: Bypass

example.com/checkout/*:
  - Cache Level: Bypass
```

## Cache Rules (New)

### Create a Cache Rule

1. Dashboard → Rules → Cache Rules
2. Create Rule

```
Expression:
(starts_with(http.request.uri.path, "/wp-content/"))

Setting: Edge TTL → Override → 7 days
Setting: Cache Level → Override → Cache Everything
```

```
Expression:
(starts_with(http.request.uri.path, "/wp-admin/") or starts_with(http.request.uri.path, "/wp-login.php"))

Setting: Cache Level → Override → Bypass
```

## Purge Cache

### Dashboard

1. Go to Dashboard → Caching → Configuration
2. Click **Purge Everything**
3. Or Custom Purge by URL/path

### API

```bash
# Purge everything
curl -X POST "https://api.cloudflare.com/client/v4/zones/$ZONE_ID/purge_cache" \
  -H "Authorization: Bearer $API_TOKEN" \
  -H "Content-Type: application/json" \
  --data '{"purge_everything": true}'

# Purge specific URLs
curl -X POST "https://api.cloudflare.com/client/v4/zones/$ZONE_ID/purge_cache" \
  -H "Authorization: Bearer $API_TOKEN" \
  -H "Content-Type: application/json" \
  --data '{
    "files": ["https://example.com/css/style.css", "https://example.com/js/app.js"]
  }'

# Purge by tag (if cache tags enabled)
curl -X POST "..." \
  --data '{"tags": ["css", "images"]}'
```

## ARGO

Argo routes traffic through the fastest network path.

### Enable

Dashboard → Speed → Optimization → Argo → Enable

### Benefits

- 30% faster load times (Cloudflare claim)
- Smart routing around network congestion
- Reduced origin server load

## Cache Reserve

Extends cache to Cloudflare's persistent storage.

```bash
# Enable via API
curl -X POST "https://api.cloudflare.com/client/v4/zones/$ZONE_ID/cache/reserve" \
  -H "Authorization: Bearer $API_TOKEN" \
  -H "Content-Type: application/json" \
  --data '{"value": "on"}'
```

## Polish (Image Optimization)

Dashboard → Speed → Optimization → Polish → Enable

| Mode | Description |
|------|-------------|
| Lossless | No quality loss, reduced size |
| Lossy | Some quality loss, smaller files |

## Auto Minify

Dashboard → Speed → Optimization → Auto Minify

Enable for: **JavaScript**, **CSS**, **HTML**

## Brotli Compression

Dashboard → Speed → Optimization → Brotli → Enable

Better compression than gzip (20-30% smaller).

## Origin Cache Control

```nginx
# In Nginx, respect Cloudflare cache headers
location ~* \.(jpg|jpeg|png|gif|ico|css|js)$ {
    expires 30d;
    add_header Cache-Control "public, immutable";
}

location / {
    # Don't cache HTML by default
    add_header Cache-Control "no-store, no-cache, must-revalidate";
}
```

## Cloudflare Workers (Custom Caching)

```js
// Custom cache behavior
addEventListener('fetch', event => {
  event.respondWith(handleRequest(event.request))
})

async function handleRequest(request) {
  const url = new URL(request.url)

  // Cache API responses for 1 hour
  if (url.pathname.startsWith('/api/')) {
    return fetch(request, {
      cf: {
        cacheTtl: 3600,
        cacheEverything: true,
      }
    })
  }

  return fetch(request)
}
```

## Cache Analytics

Monitor cache performance:

- Dashboard → Analytics & Logs → Cache
- **Hit ratio** — percentage served from cache
- **Bandwidth saved** — origin offload
- **Top uncached URLs** — what to add to cache rules

## Verification

- [ ] Static assets cached (CSS, JS, images)
- [ ] Admin/login bypassed
- [ ] Cache rules vs page rules not conflicting
- [ ] Hit ratio > 60% (good) / > 80% (excellent)
- [ ] Argo enabled
- [ ] Polish configured
- [ ] Browser cache TTL set
