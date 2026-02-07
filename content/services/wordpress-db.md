---
service: MariaDB
status: running
host: 192.168.144.41
ct: 128
port: 3306
tags: [database, mysql, mariadb]
updated: 2026-02-06
---

# WordPress Database (MariaDB)

Shared MariaDB database server for all WordPress sites.

## Overview

MariaDB server dedicated to WordPress databases. Each WordPress site has its own database and user.

**Container:** CT 128
**IP:** 192.168.144.41
**Port:** 3306
**Purpose:** Shared database for WordPress sites

## Source Code

**Docker Compose & Install Script:** [github.com/opajanvv/homelab-docker/tree/main/wordpress-db](https://github.com/opajanvv/homelab-docker/tree/main/wordpress-db)
**Local copy:** `~/dev/homelab-docker/wordpress-db/`

## Deployment

```bash
# Clone LXC template
pct clone 902 128 --hostname wordpress-db --full
pct set 128 --cores 1 --memory 1024
pct set 128 -net0 name=eth0,bridge=vmbr0,firewall=1,gw=192.168.144.1,ip=192.168.144.41/23
pct set 128 -mp0 /lxcdata/wordpress-db,mp=/data
pct set 128 -features nesting=1,keyctl=1
pct set 128 -onboot 1

# Add AppArmor workaround
cat >> /etc/pve/lxc/128.conf << 'EOF'
lxc.apparmor.profile: unconfined
lxc.mount.entry: /dev/null sys/module/apparmor/parameters/enabled none bind 0 0
EOF

# Deploy
pct start 128
pct exec 128 -- bash -c 'systemctl enable --now docker'
pct exec 128 -- bash -c 'git clone https://github.com/opajanvv/homelab-docker.git /opt/homelab-docker'
pct exec 128 -- bash -c 'cd /opt/homelab-docker/wordpress-db && chmod +x install.sh && ./install.sh'
```

## Configuration

**Environment Variables:**
- `MYSQL_ROOT_PASSWORD` - Root password (set in .env, not in repo)
- Databases created by `init.sql` on first run

**Data Locations:**
- `/data/mysql/` - Database files
- `/opt/homelab-docker/wordpress-db/` - Docker Compose config, init.sql

## Databases

Each WordPress site has its own database:

| Database | User | WordPress Site |
|----------|------|----------------|
| `wordpress` | `wordpress` | jokegoudriaan.nl (CT 129) |
| `wordpress_kledingruil` | `wp_kledingruil` | kledingruil (CT 130) |
| `wordpress_pgh` | `pgh` | pgh.janvv.nl (CT 131) |

**Password setup:** After deployment, set user passwords manually:
```sql
ALTER USER 'wordpress'@'%' IDENTIFIED BY 'password-here';
ALTER USER 'wp_kledingruil'@'%' IDENTIFIED BY 'password-here';
ALTER USER 'pgh'@'%' IDENTIFIED BY 'password-here';
FLUSH PRIVILEGES;
```

## Access

**From WordPress containers:**
- Host: `192.168.144.41:3306`
- Authentication: User + password (not in repo)

**Direct access (for admin):**
```bash
pct exec 128 -- bash -c 'docker exec -it mariadb mysql -u root -p'
```

## Backup

**Database dump (all databases):**
```bash
pct exec 128 -- bash -c 'docker exec mariadb mysqldump -u root -p --all-databases' > mariadb-backup.sql
```

**Database dump (single database):**
```bash
pct exec 128 -- bash -c 'docker exec mariadb mysqldump -u root -p wordpress' > wordpress-backup.sql
```

**Restore:**
```bash
cat mariadb-backup.sql | pct exec 128 -- bash -c 'docker exec -i mariadb mysql -u root -p'
```

**Full data backup:**
```bash
rsync -av /lxcdata/wordpress-db/ /backup/homelab/wordpress-db/
```

## Maintenance

**Update:**
```bash
pct exec 128 -- bash -c 'cd /opt/homelab-docker/wordpress-db && docker compose pull && docker compose up -d'
```

**View logs:**
```bash
pct exec 128 -- bash -c 'cd /opt/homelab-docker/wordpress-db && docker compose logs -f'
```

## Common Tasks

**Create new database:**
```sql
CREATE DATABASE wordpress_newsite;
CREATE USER 'wp_newsite'@'%' IDENTIFIED BY 'password';
GRANT ALL PRIVILEGES ON wordpress_newsite.* TO 'wp_newsite'@'%';
FLUSH PRIVILEGES;
```

**Reset user password:**
```sql
ALTER USER 'username'@'%' IDENTIFIED BY 'new-password';
FLUSH PRIVILEGES;
```

## Related

- [[services/wordpress-jokegoudriaan|WordPress - jokegoudriaan]]
- [[services/wordpress-kledingruil|WordPress - kledingruil]]
- [[services/wordpress-pgh|WordPress - pgh]]
- [[how-to/backup-restore|Backup & Restore]]
