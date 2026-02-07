---
service: n8n
status: running
host: 192.168.144.61
ct: 120
port: 5678
tags: [automation, workflow]
updated: 2026-02-06
---

# n8n

Workflow automation platform. Connect services and automate tasks.

## Overview

n8n is a workflow automation tool similar to Zapier or Make, but self-hosted. Used for automating tasks between services.

**Access:** https://n8n.janvv.nl
**Container:** CT 120
**IP:** 192.168.144.61
**Port:** 5678

## Source Code

**Docker Compose & Install Script:** [github.com/opajanvv/homelab-docker/tree/main/n8n](https://github.com/opajanvv/homelab-docker/tree/main/n8n)
**Local copy:** `~/dev/homelab-docker/n8n/`

## Deployment

```bash
# Clone LXC template
pct clone 902 120 --hostname n8n --full
pct set 120 --cores 1 --memory 1024
pct set 120 -net0 name=eth0,bridge=vmbr0,firewall=1,gw=192.168.144.1,ip=192.168.144.61/23
pct set 120 -mp0 /lxcdata/n8n,mp=/data
pct set 120 -features nesting=1,keyctl=1
pct set 120 -onboot 1

# Add AppArmor workaround to /etc/pve/lxc/120.conf
cat >> /etc/pve/lxc/120.conf << 'EOF'
lxc.apparmor.profile: unconfined
lxc.mount.entry: /dev/null sys/module/apparmor/parameters/enabled none bind 0 0
EOF

# Deploy
pct start 120
pct exec 120 -- bash -c 'systemctl enable --now docker'
pct exec 120 -- bash -c 'git clone https://github.com/opajanvv/homelab-docker.git /opt/homelab-docker'
pct exec 120 -- bash -c 'cd /opt/homelab-docker/n8n && chmod +x install.sh && ./install.sh'
```

## Configuration

**Environment Variables:**
- `N8N_HOST` - Domain for n8n
- `N8N_PORT` - Port (default: 5678)
- `N8N_PROTOCOL` - http
- `WEBHOOK_URL` - Full URL for webhooks
- `N8N_ENCRYPTION_KEY` - Encryption key for credentials

**Data Location:**
- `/data/n8n/` - Workflow definitions, credentials
- `/opt/homelab-docker/n8n/` - Docker Compose config

## Access

- **Web UI:** https://n8n.janvv.nl
- **Direct:** http://192.168.144.61:5678

## Backup

**What to backup:**
- `/lxcdata/n8n/` - All n8n data (workflows, credentials)

**Backup command:**
```bash
rsync -av /lxcdata/n8n/ /backup/homelab/n8n/
```

**Restore:**
```bash
# Stop container
pct stop 120

# Restore data
rsync -av /backup/homelab/n8n/ /lxcdata/n8n/

# Start container
pct start 120
```

## Maintenance

**Update:**
```bash
pct exec 120 -- bash -c 'cd /opt/homelab-docker/n8n && docker compose pull && docker compose up -d'
```

**View logs:**
```bash
pct exec 120 -- bash -c 'cd /opt/homelab-docker/n8n && docker compose logs -f'
```

**Restart:**
```bash
pct exec 120 -- bash -c 'cd /opt/homelab-docker/n8n && docker compose restart'
```

## Common Tasks

**Create workflow:** Via web UI
**Import/Export workflows:** Via web UI (Settings → Workflows)
**Check execution history:** Via web UI (Executions)

## Related

- [[infrastructure/network|Network routing]]
- [[how-to/update-service|Update a Service]]
- [[how-to/backup-restore|Backup & Restore]]
