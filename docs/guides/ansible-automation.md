# Ansible Automation

## Installation

```bash
# Ubuntu / Debian
apt update && apt install -y ansible

# Using pip
pip install ansible

# Verify
ansible --version
```

## Inventory File

```ini title="inventory.ini"
[web]
web1 ansible_host=192.168.1.10 ansible_user=root
web2 ansible_host=192.168.1.11 ansible_user=root

[db]
db1 ansible_host=192.168.1.20 ansible_user=root

[all:vars]
ansible_ssh_private_key_file=~/.ssh/id_ed25519
```

### Group Variables

```yaml title="group_vars/all.yml"
ntp_server: pool.ntp.org
admin_email: admin@example.com
```

```yaml title="group_vars/web.yml"
nginx_port: 443
php_version: 8.2
```

```yaml title="host_vars/web1.yml"
domain: example.com
```

## Ad-Hoc Commands

```bash
# Ping all hosts
ansible all -i inventory.ini -m ping

# Run command
ansible all -i inventory.ini -m shell -a "uptime"

# Install package
ansible web -i inventory.ini -m apt -a "name=nginx state=present" -b

# Copy file
ansible all -i inventory.ini -m copy -a "src=./config.conf dest=/etc/nginx/conf.d/" -b

# Service management
ansible web -i inventory.ini -m service -a "name=nginx state=restarted" -b
```

## Playbook Structure

```yaml title="site.yml"
---
- name: Apply common configuration
  hosts: all
  become: yes
  roles:
    - common

- name: Configure web servers
  hosts: web
  become: yes
  roles:
    - nginx
    - php

- name: Configure database servers
  hosts: db
  become: yes
  roles:
    - mysql
```

## Sample Playbook: Server Hardening

```yaml title="playbooks/harden-server.yml"
---
- name: Harden server
  hosts: all
  become: yes
  vars:
    ssh_port: 22
    admin_user: devops

  tasks:
    - name: Update apt cache
      apt:
        update_cache: yes
        cache_valid_time: 3600

    - name: Install security packages
      apt:
        name:
          - fail2ban
          - ufw
          - unattended-upgrades
        state: present

    - name: Create admin user
      user:
        name: "{{ admin_user }}"
        groups: sudo
        shell: /bin/bash
        create_home: yes

    - name: Set SSH key
      authorized_key:
        user: "{{ admin_user }}"
        key: "{{ lookup('file', '~/.ssh/id_ed25519.pub') }}"

    - name: Harden SSH
      lineinfile:
        path: /etc/ssh/sshd_config
        regexp: "{{ item.regexp }}"
        line: "{{ item.line }}"
      loop:
        - { regexp: "^PermitRootLogin", line: "PermitRootLogin no" }
        - { regexp: "^PasswordAuthentication", line: "PasswordAuthentication no" }
        - { regexp: "^PubkeyAuthentication", line: "PubkeyAuthentication yes" }
      notify: restart sshd

    - name: Configure UFW
      ufw:
        rule: "{{ item.rule }}"
        port: "{{ item.port }}"
        proto: tcp
      loop:
        - { rule: allow, port: "22" }
        - { rule: allow, port: "80" }
        - { rule: allow, port: "443" }
      notify: enable ufw

  handlers:
    - name: restart sshd
      service:
        name: sshd
        state: restarted

    - name: enable ufw
      ufw:
        state: enabled
```

## Sample Playbook: Nginx + PHP

```yaml title="playbooks/webserver.yml"
---
- name: Setup web server
  hosts: web
  become: yes
  vars:
    domain: example.com
    php_version: "8.2"

  tasks:
    - name: Install Nginx
      apt:
        name: nginx
        state: present

    - name: Install PHP
      apt:
        name:
          - "php{{ php_version }}-fpm"
          - "php{{ php_version }}-mysql"
          - "php{{ php_version }}-cli"
        state: present

    - name: Copy Nginx config
      template:
        src: templates/nginx.conf.j2
        dest: "/etc/nginx/sites-available/{{ domain }}"

    - name: Enable site
      file:
        src: "/etc/nginx/sites-available/{{ domain }}"
        dest: "/etc/nginx/sites-enabled/{{ domain }}"
        state: link
      notify: reload nginx

    - name: Create web root
      file:
        path: "/var/www/{{ domain }}"
        state: directory
        owner: www-data
        group: www-data
        mode: "0755"

  handlers:
    - name: reload nginx
      service:
        name: nginx
        state: reloaded
```

## Role Structure

```
roles/
├── nginx/
│   ├── tasks/
│   │   └── main.yml
│   ├── handlers/
│   │   └── main.yml
│   ├── templates/
│   │   └── nginx.conf.j2
│   ├── files/
│   │   └── default.conf
│   └── vars/
│       └── main.yml
├── php/
└── mysql/
```

## Jinja2 Template Example

```nginx title="templates/nginx.conf.j2"
server {
    listen 80;
    server_name {{ domain }} www.{{ domain }};
    return 301 https://{{ domain }}$request_uri;
}

server {
    listen 443 ssl http2;
    server_name {{ domain }};

    root /var/www/{{ domain }}/public;
    index index.php;

    ssl_certificate /etc/letsencrypt/live/{{ domain }}/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/{{ domain }}/privkey.pem;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location ~ \.php$ {
        fastcgi_pass unix:/var/run/php/php{{ php_version }}-fpm.sock;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
    }
}
```

## Running Playbooks

```bash
# Run specific playbook
ansible-playbook -i inventory.ini playbooks/harden-server.yml

# Check mode (dry run)
ansible-playbook -i inventory.ini playbooks/harden-server.yml --check

# Verbose
ansible-playbook -i inventory.ini playbooks/harden-server.yml -v

# Limit to specific host
ansible-playbook -i inventory.ini playbooks/harden-server.yml --limit web1

# Ask for sudo password
ansible-playbook -i inventory.ini playbooks/harden-server.yml -K

# Tag-specific tasks
ansible-playbook -i inventory.ini playbooks/harden-server.yml --tags ssh,ufw
```

## Ansible Vault

```bash
# Encrypt file
ansible-vault encrypt secrets.yml

# View encrypted file
ansible-vault view secrets.yml

# Edit encrypted file
ansible-vault edit secrets.yml

# Use vault password file
ansible-playbook -i inventory.ini playbooks/deploy.yml --vault-password-file .vault_pass
```

## Useful Modules

| Module | Purpose |
|--------|---------|
| `apt` / `yum` | Package management |
| `copy` | Copy files to remote |
| `template` | Jinja2 template rendering |
| `service` / `systemd` | Service management |
| `user` | User management |
| `lineinfile` | Edit config files |
| `replace` | Regex-based file editing |
| `file` | File/directory operations |
| `git` | Git repository management |
| `docker_container` | Docker container management |
| `docker_compose` | Docker compose management |
| `mysql_db` / `mysql_user` | MySQL management |
| `ufw` | Firewall management |
| `certbot` | SSL certificate management |
| `cron` | Cron job management |

## Verification

- [ ] Inventory file defines all servers
- [ ] Playbooks run in check mode without errors
- [ ] Secrets encrypted with Ansible Vault
- [ ] Roles organized by concern
- [ ] Templates use variables, not hardcoded values
- [ ] Handlers for service reloads
