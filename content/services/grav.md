---
service: Grav
status: running
host: 192.168.144.72
ct: 123
port: 80
tags: [web, cms, flat-file]
updated: 2026-02-06
---

# Grav

Flat-file CMS for personal websites.

## Overview

Grav is a flat-file CMS meaning no database - all content is stored as files. Used for opa.janvv.nl and www.janvv.nl.

**Access:** https://opa.janvv.nl, https://www.janvv.nl
**Container:** CT 123
**IP:** 192.168.144.72
**Port:** 80

## Source Code

**Docker Compose & Install Script:** [github.com/opajanvv/homelab-docker/tree/main/grav](https://github.com/opajanvv/homelab-docker/tree/main/grav)
**Local copy:** `~/dev/homelab-docker/grav/`

## Deployment

```bash
# Clone LXC template
pct clone 902 123 --hostname grav --full
pct set 123 --cores 1 --memory 512
pct set 123 -net0 name=eth0,bridge=vmbr0,firewall=1,gw=192.168.144.1,ip=192.168.144.72/23
pct set 123 -mp0 /lxcdata/grav,mp=/data
pct set 123 -features nesting=1,keyctl=1
pct set 123 -onboot 1

# Add AppArmor workaround
cat >> /etc/pve/lxc/123.conf << 'EOF'
lxc.apparmor.profile: unconfined
lxc.mount.entry: /dev/null sys/module/apparmor/parameters/enabled none bind 0 0
EOF

# Deploy
pct start 123
pct exec 123 -- bash -c 'systemctl enable --now docker'
pct exec 123 -- bash -c 'git clone https://github.com/opajanvv/homelab-docker.git /opt/homelab-docker'
pct exec 123 -- bash -c 'cd /opt/homelab-docker/grav && chmod +x install.sh && ./install.sh'
```

## Configuration

**Stack:** Nginx + PHP-FPM

**Data Locations:**
- `/data/html/` - Grav installation and content
- `/opt/homelab-docker/grav/` - Docker Compose config, nginx.conf

**Nginx Config:** Custom nginx.conf for handling Grav's routing

## Access

- **opa.janvv.nl:** https://opa.janvv.nl
- **www.janvv.nl:** https://www.janvv.nl
- **Direct:** http://192.168.144.72

## Content Management

**Editing content:** Content is in Markdown files under `/data/html/user/pages/`
- Login to admin panel: https://opa.janvv.nl/admin
- Edit pages via web admin or by editing Markdown files directly

**Themes and plugins:** Managed via Grav admin or by copying to respective directories

## Backup

**What to backup:**
- `/lxcdata/grav/html/` - Entire Grav installation including content

**Backup command:**
```bash
rsync -av /lxcdata/grav/html/ /backup/homelab/grav/
```

**Restore:**
```bash
pct stop 123
rsync -av /backup/homelab/grav/ /lxcdata/grav/html/
pct start 123
```

## Maintenance

**Update:**
```bash
pct exec 123 -- bash -c 'cd /opt/homelab-docker/grav && docker compose pull && docker compose up -d'
```

**Update Grav (via admin):** Admin → Updates
**Clear cache:** Admin → Tools → Clear Cache

**View logs:**
```bash
pct exec 123 -- bash -c 'cd /opt/homelab-docker/grav && docker compose logs -f'
```

## Common Tasks

**Create page:** Admin → Pages → Add Page
**Install plugin:** Admin → Plugins → Add
**Install theme:** Admin → Themes → Add

## Related

- [[infrastructure/network|Network routing]]
- [[how-to/update-service|Update a Service]]
- [[how-to/backup-restore|Backup & Restore]]
