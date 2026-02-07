---
service: WordPress
status: running
host: 192.168.144.73
ct: 131
port: 80
tags: [web, cms, wordpress]
updated: 2026-02-06
site: pgh.janvv.nl
---

# WordPress - pgh.janvv.nl

Personal WordPress site.

## Overview

Standard WordPress installation using shared MariaDB database.

**Access:** https://pgh.janvv.nl
**Container:** CT 131
**IP:** 192.168.144.73
**Port:** 80

## Source Code

**Docker Compose & Install Script:** [github.com/opajanvv/homelab-docker/tree/main/wordpress-pgh](https://github.com/opajanvv/homelab-docker/tree/main/wordpress-pgh)
**Local copy:** `~/dev/homelab-docker/wordpress-pgh/`

## Deployment

```bash
# Clone LXC template
pct clone 902 131 --hostname wordpress-pgh --full
pct set 131 --cores 1 --memory 1024
pct set 131 -net0 name=eth0,bridge=vmbr0,firewall=1,gw=192.168.144.1,ip=192.168.144.73/23
pct set 131 -mp0 /lxcdata/wordpress-pgh,mp=/data
pct set 131 -features nesting=1,keyctl=1
pct set 131 -onboot 1

# Add AppArmor workaround
cat >> /etc/pve/lxc/131.conf << 'EOF'
lxc.apparmor.profile: unconfined
lxc.mount.entry: /dev/null sys/module/apparmor/parameters/enabled none bind 0 0
EOF

# Deploy
pct start 131
pct exec 131 -- bash -c 'systemctl enable --now docker'
pct exec 131 -- bash -c 'git clone https://github.com/opajanvv/homelab-docker.git /opt/homelab-docker'
pct exec 131 -- bash -c 'cd /opt/homelab-docker/wordpress-pgh && chmod +x install.sh && ./install.sh'
```

## Configuration

**Stack:** Nginx + PHP-FPM

**Database:**
- Host: `192.168.144.41:3306` (shared MariaDB)
- Database: `wordpress_pgh`
- User: `pgh`
- Password: Set in `.env` (not in repo)

**Data Locations:**
- `/data/html/` - WordPress files
- `/opt/homelab-docker/wordpress-pgh/` - Docker Compose config

## Access

- **Public:** https://pgh.janvv.nl
- **Admin:** https://pgh.janvv.nl/wp-admin/
- **Direct:** http://192.168.144.73

## Backup

**What to backup:**
- `/lxcdata/wordpress-pgh/html/` - WordPress files
- Database: See [[services/wordpress-db|MariaDB backup]]

**Backup command:**
```bash
rsync -av /lxcdata/wordpress-pgh/html/ /backup/homelab/wordpress-pgh/
```

**Database backup:**
```bash
pct exec 128 -- bash -c 'docker exec mariadb mysqldump -u root -p wordpress_pgh' > /backup/homelab/wordpress-pgh.sql
```

## Maintenance

**Update WordPress:** Via admin dashboard
**Update Docker:**
```bash
pct exec 131 -- bash -c 'cd /opt/homelab-docker/wordpress-pgh && docker compose pull && docker compose up -d'
```

**View logs:**
```bash
pct exec 131 -- bash -c 'cd /opt/homelab-docker/wordpress-pgh && docker compose logs -f'
```

## Related

- [[services/wordpress-db|Shared MariaDB]]
- [[services/wordpress-jokegoudriaan|WordPress - jokegoudriaan]]
- [[services/wordpress-kledingruil|WordPress - kledingruil]]
