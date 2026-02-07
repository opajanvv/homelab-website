---
service: WordPress
status: running
host: 192.168.144.71
ct: 130
port: 80
tags: [web, cms, wordpress]
updated: 2026-02-06
site: kledingruil.jokegoudriaan.nl
---

# WordPress - kledingruil.jokegoudriaan.nl

WordPress site for kledingruil subdomain.

## Overview

Standard WordPress installation using shared MariaDB database.

**Access:** https://kledingruil.jokegoudriaan.nl
**Container:** CT 130
**IP:** 192.168.144.71
**Port:** 80

## Source Code

**Docker Compose & Install Script:** [github.com/opajanvv/homelab-docker/tree/main/wordpress-kledingruil](https://github.com/opajanvv/homelab-docker/tree/main/wordpress-kledingruil)
**Local copy:** `~/dev/homelab-docker/wordpress-kledingruil/`

## Deployment

```bash
# Clone LXC template
pct clone 902 130 --hostname wordpress-kledingruil --full
pct set 130 --cores 1 --memory 1024
pct set 130 -net0 name=eth0,bridge=vmbr0,firewall=1,gw=192.168.144.1,ip=192.168.144.71/23
pct set 130 -mp0 /lxcdata/wordpress-kledingruil,mp=/data
pct set 130 -features nesting=1,keyctl=1
pct set 130 -onboot 1

# Add AppArmor workaround
cat >> /etc/pve/lxc/130.conf << 'EOF'
lxc.apparmor.profile: unconfined
lxc.mount.entry: /dev/null sys/module/apparmor/parameters/enabled none bind 0 0
EOF

# Deploy
pct start 130
pct exec 130 -- bash -c 'systemctl enable --now docker'
pct exec 130 -- bash -c 'git clone https://github.com/opajanvv/homelab-docker.git /opt/homelab-docker'
pct exec 130 -- bash -c 'cd /opt/homelab-docker/wordpress-kledingruil && chmod +x install.sh && ./install.sh'
```

## Configuration

**Stack:** Nginx + PHP-FPM

**Database:**
- Host: `192.168.144.41:3306` (shared MariaDB)
- Database: `wordpress_kledingruil`
- User: `wp_kledingruil`
- Password: Set in `.env` (not in repo)

**Data Locations:**
- `/data/html/` - WordPress files
- `/opt/homelab-docker/wordpress-kledingruil/` - Docker Compose config

## Access

- **Public:** https://kledingruil.jokegoudriaan.nl
- **Admin:** https://kledingruil.jokegoudriaan.nl/wp-admin/
- **Direct:** http://192.168.144.71

## Backup

**What to backup:**
- `/lxcdata/wordpress-kledingruil/html/` - WordPress files
- Database: See [[services/wordpress-db|MariaDB backup]]

**Backup command:**
```bash
rsync -av /lxcdata/wordpress-kledingruil/html/ /backup/homelab/wordpress-kledingruil/
```

**Database backup:**
```bash
pct exec 128 -- bash -c 'docker exec mariadb mysqldump -u root -p wordpress_kledingruil' > /backup/homelab/wordpress-kledingruil.sql
```

## Maintenance

**Update WordPress:** Via admin dashboard
**Update Docker:**
```bash
pct exec 130 -- bash -c 'cd /opt/homelab-docker/wordpress-kledingruil && docker compose pull && docker compose up -d'
```

**View logs:**
```bash
pct exec 130 -- bash -c 'cd /opt/homelab-docker/wordpress-kledingruil && docker compose logs -f'
```

## Related

- [[services/wordpress-db|Shared MariaDB]]
- [[services/wordpress-jokegoudriaan|WordPress - jokegoudriaan]]
- [[services/wordpress-pgh|WordPress - pgh]]
