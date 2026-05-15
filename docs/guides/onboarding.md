---
tags:
  - onboarding
  - setup
---

# Onboarding Guide

## Prerequisites

- GitHub account
- Access to company VPN
- 1Password / LastPass account

## Step 1: Tool Installation

```bash
# Install essential tools
brew install git kubectl helm terraform awscli
brew install --cask docker visual-studio-code iterm2

# Install Kubernetes tools
brew install kind k9s istioctl velero
```

## Step 2: Configure Access

1. Request GitHub team access from your manager
2. Configure kubectl:
   ```bash
   aws eks update-kubeconfig --region us-east-1 --name cluster-name
   ```
3. Verify access:
   ```bash
   kubectl get nodes
   kubectl get pods -A
   ```

## Step 3: Clone Repositories

```bash
mkdir ~/devops && cd ~/devops
git clone https://github.com/example/wl-devops-docs.git
git clone https://github.com/example/infrastructure.git
git clone https://github.com/example/deployment.git
```
