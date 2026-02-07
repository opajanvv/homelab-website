---
service: WordPress
status: running
host: 192.168.144.70
ct: 129
port: 80
tags: [web, cms, wordpress]
updated: 2026-02-06
site: jokegoudriaan.nl
---

# WordPress - jokegoudriaan.nl

WordPress site for jokegoudriaan.nl client.

## Overview

Standard WordPress installation using shared MariaDB database.

**Access:** https://jokegoudriaan.nl
**Container:** CT 129
**IP:** 192.168.144.70
**Port:** 80

## Source Code

**Docker Compose & Install Script:** [github.com/opajanvv/homelab-docker/tree/main/wordpress-jokegoudriaan](https://github.com/opajanvv/homelab-docker/tree/main/wordpress-jokegoudriaan)
**Local copy:** `~/dev/homelab-docker/wordpress-jokegoudriaan/`

## Deployment

```bash
# Clone LXC template
pct clone 902 129 --hostname wordpress-jokegoudriaan --full
pct set 129 --cores 1 --memory 1024
pct set 129 -net0 name=eth0,bridge=vmbr0,firewall=1,gw=192.168.144.1,ip=192.168.144.70/23
pct set 129 -mp0 /lxcdata/wordpress-jokegoudriaan,mp=/data
pct set 129 -features nesting=1,keyctl=1
pct set 129 -onboot 1

# Add AppArmor workaround
cat >> /etc/pve/lxc/129.conf << 'EOF'
lxc.apparmor.profile: unconfined
lxc.mount.entry: /dev/null sys/module/apparmor/parameters/enabled none bind 0 0
EOF

# Deploy
pct start 129
pct exec 129 -- bash -c 'systemctl enable --now docker'
pct exec 129 -- bash -c 'git clone https://github.com/opajanvv/homelab-docker.git /opt/homelab-docker'
pct exec 129 -- bash -c 'cd /opt/homelab-docker/wordpress-jokegoudriaan && chmod +x install.sh && ./install.sh'
```

## Configuration

**Stack:** Nginx + PHP-FPM

**Database:**
- Host: `192.168.144.41:3306` (shared MariaDB)
- Database: `wordpress`
- User: `wordpress`
- Password: Set in `.env` (not in repo)

**Data Locations:**
- `/data/html/` - WordPress files
- `/opt/homelab-docker/wordpress-jokegoudriaan/` - Docker Compose config

## Access

- **Public:** https://jokegoudriaan.nl
- **Admin:** https://jokegoudriaan.nl/wp-admin/
- **Direct:** http://192.168.144.70

## Backup

**What to backup:**
- `/lxcdata/wordpress-jokegoudriaan/html/` - WordPress files, themes, plugins, uploads
- Database: See [[services/wordpress-db|MariaDB backup]]

**Backup command:**
```bash
rsync -av /lxcdata/wordpress-jokegoudriaan/html/ /backup/homelab/wordpress-jokegoudriaan/
```

**Full backup (files + DB):**
```bash
# Files
rsync -av /lxcdata/wordpress-jokegoudriaan/html/ /backup/homelab/wordpress-jokegoudriaan/

# Database
pct exec 128 -- bash -c 'docker exec mariadb mysqldump -u root -p wordpress' > /backup/homelab/wordpress-jokegoudriaan.sql
```

## Maintenance

**Update WordPress:** Via admin dashboard (Dashboard → Updates)
**Update themes/plugins:** Via admin dashboard or manually

**Update Docker containers:**
```bash
pct exec 129 -- bash -c 'cd /opt/homelab-docker/wordpress-jokegoudriaan && docker compose pull && docker compose up -d'
```

**View logs:**
```bash
pct exec 129 -- bash -c 'cd /opt/homelab-docker/wordpress-jokegoudriaan && docker compose logs -f'
```

## Common Tasks

**Install plugin:** Admin → Plugins → Add New
**Install theme:** Admin → Appearance → Add New
**Update permalink settings:** Admin → Settings → Permalinks (save after changes)

## Related

- [[services/wordpress-db|Shared MariaDB]]
- [[services/wordpress-kledingruil|WordPress - kledingruil]]
- [[services/wordpress-pgh|WordPress - pgh]]
- [[infrastructure/network|Network routing]]
