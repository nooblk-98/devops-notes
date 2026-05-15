# Node.js Installation

Install Node.js runtime and npm on Linux servers. Covers multiple methods including package managers, NVM, and Docker.

## Option 1: Using NodeSource Repository

Supports pinned versions across server rebuilds. Best for production servers.

### Ubuntu / Debian

```bash
# Install Node.js 20 LTS
curl -fsSL https://deb.nodesource.com/setup_20.x | bash -
apt install -y nodejs

# Verify
node --version
npm --version
```

### RHEL / Rocky / Alma

```bash
# Install Node.js 20 LTS
curl -fsSL https://rpm.nodesource.com/setup_20.x | bash -
dnf install -y nodejs

# Verify
node --version
npm --version
```

### Available Versions

Replace `20.x` in the URL with your target version:

| Version | Status | URL |
|---------|--------|-----|
| 22.x | Current | `https://deb.nodesource.com/setup_22.x` |
| 20.x | LTS | `https://deb.nodesource.com/setup_20.x` |
| 18.x | EOL | `https://deb.nodesource.com/setup_18.x` |

## Option 2: Using NVM (Node Version Manager)

Best for development environments where you need to switch between Node versions frequently.

### Install NVM

```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.4/install.sh | bash

# Reload shell
source ~/.bashrc
```

### Install and Switch Node Versions

```bash
# Install latest LTS
nvm install --lts

# Install specific version
nvm install 22
nvm install 18

# Switch versions
nvm use 22
nvm use --lts

# Set default
nvm alias default 20

# List installed
nvm list
```

### Per-Project .nvmrc

Create `.nvmrc` in your project root:

```
20
```

Then use `nvm use` to auto-switch when entering the directory (add to shell rc):

```bash
# ~/.bashrc
cd() { builtin cd "$@" && [ -f .nvmrc ] && nvm use; }
```

## Option 3: Docker Installation

### Using Official Node Image

```bash
# Run Node container interactively
docker run -it --rm node:20-alpine node -e "console.log('Hello from Node ' + process.version)"

# Run your app
docker run -d \
  --name my-app \
  --restart unless-stopped \
  -p 3000:3000 \
  -v $(pwd):/app \
  -w /app \
  node:20-alpine node server.js
```

### Docker Compose for Node.js App

```yaml
services:
  app:
    image: node:20-alpine
    container_name: my-app
    restart: unless-stopped
    ports:
      - "3000:3000"
    volumes:
      - .:/app
    working_dir: /app
    command: sh -c "npm install && node server.js"
    environment:
      - NODE_ENV=production
```

### Multi-stage Build for Production

```dockerfile
# Build stage
FROM node:20-alpine AS build
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production

# Run stage
FROM node:20-alpine
WORKDIR /app
COPY --from=build /app/node_modules ./node_modules
COPY . .
USER node
EXPOSE 3000
CMD ["node", "server.js"]
```

## Option 4: Binary Download (No Package Manager)

Useful for air-gapped environments or custom installation paths.

```bash
# Download and extract
wget https://nodejs.org/dist/v20.19.1/node-v20.19.1-linux-x64.tar.xz
tar xf node-v20.19.1-linux-x64.tar.xz
mv node-v20.19.1-linux-x64 /usr/local/node

# Add to PATH
echo 'export PATH=/usr/local/node/bin:$PATH' | tee /etc/profile.d/node.sh
chmod +x /etc/profile.d/node.sh
source /etc/profile.d/node.sh

# Verify
node --version
npm --version
```

## Install Build Tools

Required for compiling native npm modules.

### Ubuntu / Debian

```bash
apt install -y build-essential python3
```

### RHEL / Rocky / Alma

```bash
dnf groupinstall -y "Development Tools"
dnf install -y python3
```

## Install PM2 (Process Manager)

```bash
npm install -g pm2

# Start app
pm2 start server.js --name my-app

# Save process list
pm2 save

# Auto-start on reboot
pm2 startup
```

## Configure npm

```bash
# Set default registry (useful for private registries)
npm config set registry https://registry.npmjs.org/

# Set prefix for global installs (avoid sudo)
mkdir -p ~/.npm-global
npm config set prefix ~/.npm-global
echo 'export PATH=~/.npm-global/bin:$PATH' >> ~/.bashrc
```

## Verify Installation

```bash
# Node version
node -e "console.log(process.version)"

# npm version
npm --version

# Corepack (comes with Node 20+)
corepack --version
```

## Docker Node Image Variants

| Tag | OS | Size | Use Case |
|-----|----|------|----------|
| `node:22-alpine` | Alpine Linux | ~130 MB | Production (smallest) |
| `node:20-alpine` | Alpine Linux | ~130 MB | Production LTS |
| `node:20-slim` | Debian slim | ~180 MB | When glibc required |
| `node:20` | Debian full | ~360 MB | Development / build |
| `node:20-bullseye-slim` | Debian 11 slim | ~180 MB | Compatibility |

## Troubleshooting

**npm ERR! code EACCES:**
```bash
# Fix permissions (do NOT use sudo with npm)
npm config set prefix ~/.npm-global
```

**node: command not found after install:**
```bash
# Re-source profile
source ~/.bashrc

# Or check PATH
echo $PATH
```

**npm ERR! gyp ERR! stack Error: not found: make:**
```bash
# Install build tools (see above)
apt install build-essential
```
