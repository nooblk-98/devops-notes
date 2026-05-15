# SSH Key Management

## Generate SSH Key Pair

```bash
# Ed25519 (recommended)
ssh-keygen -t ed25519 -C "your_email@example.com"

# RSA (for older systems)
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"

# Specify file location
ssh-keygen -t ed25519 -f ~/.ssh/mykey -C "description"
```

## Deploy Public Key to Server

```bash
# Using ssh-copy-id (recommended)
ssh-copy-id user@server-ip

# Manually
cat ~/.ssh/id_ed25519.pub | ssh user@server-ip "mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys"

# From local file
ssh user@server-ip "echo '$(cat ~/.ssh/id_ed25519.pub)' >> ~/.ssh/authorized_keys"
```

## SSH Config File

```ini title="~/.ssh/config"
# Default
Host *
    ServerAliveInterval 60
    ServerAliveCountMax 3

# Specific server
Host myserver
    HostName 192.168.1.100
    User devops
    Port 22
    IdentityFile ~/.ssh/mykey
    ForwardAgent yes

# Group of servers
Host *.example.com
    User admin
    IdentityFile ~/.ssh/example_key
```

Usage: `ssh myserver`

## Key Permissions

```bash
# SSH directory
chmod 700 ~/.ssh

# Private key
chmod 600 ~/.ssh/id_ed25519
chmod 600 ~/.ssh/mykey

# Public key
chmod 644 ~/.ssh/id_ed25519.pub

# Config
chmod 600 ~/.ssh/config

# Authorized keys
chmod 600 ~/.ssh/authorized_keys
```

## Multiple Keys for Different Services

```bash
# GitHub personal
ssh-keygen -t ed25519 -f ~/.ssh/github_personal -C "personal@email.com"

# GitHub work
ssh-keygen -t ed25519 -f ~/.ssh/github_work -C "work@email.com"

# Server access
ssh-keygen -t ed25519 -f ~/.ssh/server_key -C "server access"
```

### Config for Multiple Keys

```ini title="~/.ssh/config"
Host github.com
    IdentityFile ~/.ssh/github_personal

Host work.github.com
    HostName github.com
    IdentityFile ~/.ssh/github_work

Host server-*
    IdentityFile ~/.ssh/server_key
```

## Add Public Key to GitHub

```bash
# Copy public key
cat ~/.ssh/id_ed25519.pub

# Or use gh CLI
gh ssh-key add ~/.ssh/id_ed25519.pub --title "My Laptop"
```

## Key Rotation

### Generate New Key

```bash
# Step 1: Generate new key
ssh-keygen -t ed25519 -f ~/.ssh/new_key -C "new_key_$(date +%Y%m)"

# Step 2: Add new public key to servers
ssh-copy-id -i ~/.ssh/new_key.pub user@server

# Step 3: Test new key
ssh -i ~/.ssh/new_key user@server

# Step 4: Remove old key from servers
ssh user@server "sed -i '/old_key/d' ~/.ssh/authorized_keys"

# Step 5: Update default key
mv ~/.ssh/id_ed25519 ~/.ssh/old_backup
mv ~/.ssh/new_key ~/.ssh/id_ed25519
mv ~/.ssh/new_key.pub ~/.ssh/id_ed25519.pub
```

## SSH Agent

```bash
# Start agent
eval "$(ssh-agent -s)"

# Add key
ssh-add ~/.ssh/id_ed25519

# Add with timeout
ssh-add -t 3600 ~/.ssh/id_ed25519  # 1 hour

# List keys
ssh-add -l

# Remove all keys
ssh-add -D
```

## Disable Password Auth on Server

```ini title="/etc/ssh/sshd_config"
PasswordAuthentication no
PubkeyAuthentication yes
ChallengeResponseAuthentication no
UsePAM no
```

```bash
systemctl reload sshd
```

## Revoke a Key

```bash
# Remove from server
ssh user@server "sed -i '/key-to-revoke/d' ~/.ssh/authorized_keys"

# Check remaining keys
ssh user@server "cat ~/.ssh/authorized_keys"

# Remove from GitHub
gh ssh-key list
gh ssh-key delete <id>
```

## Troubleshooting

```bash
# Test connection with verbose
ssh -vvv user@server

# Check which keys are offered
ssh-add -l

# Check server logs for auth failures
ssh user@server "journalctl -u sshd | grep 'Failed' | tail -10"

# Check authorized_keys format
ssh user@server "cat ~/.ssh/authorized_keys"

# Permission issues on server
ssh user@server "ls -la ~/.ssh/"
```

## Best Practices

- Use Ed25519 keys (faster, more secure than RSA)
- Always use passphrase on private keys
- Rotate keys every 6-12 months
- Never share private keys
- Use different keys for different services
- Revoke keys when team members leave
- Audit keys quarterly
