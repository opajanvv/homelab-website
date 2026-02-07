---
service: Planka
status: running
host: 192.168.144.60
ct: 122
port: 1337
tags: [app, project-management, kanban]
updated: 2026-02-06
---

# Planka

Project management and Kanban board application.

## Overview

Planka is an open-source project management tool with Kanban boards, similar to Trello.

**Access:** https://tasks.janvv.nl
**Container:** CT 122
**IP:** 192.168.144.60
**Port:** 1337

## Source Code

**Docker Compose & Install Script:** [github.com/opajanvv/homelab-docker/tree/main/planka](https://github.com/opajanvv/homelab-docker/tree/main/planka)
**Local copy:** `~/dev/homelab-docker/planka/`

## Deployment

```bash
# Clone LXC template
pct clone 902 122 --hostname planka --full
pct set 122 --cores 1 --memory 1024
pct set 122 -net0 name=eth0,bridge=vmbr0,firewall=1,gw=192.168.144.1,ip=192.168.144.60/23
pct set 122 -mp0 /lxcdata/planka,mp=/data
pct set 122 -features nesting=1,keyctl=1
pct set 122 -onboot 1

# Add AppArmor workaround
cat >> /etc/pve/lxc/122.conf << 'EOF'
lxc.apparmor.profile: unconfined
lxc.mount.entry: /dev/null sys/module/apparmor/parameters/enabled none bind 0 0
EOF

# Deploy
pct start 122
pct exec 122 -- bash -c 'systemctl enable --now docker'
pct exec 122 -- bash -c 'git clone https://github.com/opajanvv/homelab-docker.git /opt/homelab-docker'
pct exec 122 -- bash -c 'cd /opt/homelab-docker/planka && chmod +x install.sh && ./install.sh'
```

## Configuration

**Stack:** PostgreSQL database + Planka application

**Environment Variables:**
- `BASE_URL` - https://tasks.janvv.nl
- `SECRET_KEY` - Application secret
- `POSTGRES_DB` - planka
- `POSTGRES_USER` - postgres
- Database uses trust authentication (no password from Docker network)

**Data Locations:**
- `/data/postgresql/` - PostgreSQL data
- `/data/planka/` - Planka application data
- `/data/attachments/` - File attachments
- `/data/project-background-images/` - Board backgrounds
- `/data/user-avatars/` - User avatars
- `/opt/homelab-docker/planka/` - Docker Compose config

## Access

- **Web UI:** https://tasks.janvv.nl
- **Direct:** http://192.168.144.60:1337

**First Visit:** Create admin account

## Backup

**What to backup:**
- `/lxcdata/planka/postgres/` - Database
- `/lxcdata/planka/avatars/` - User data
- `/lxcdata/planka/background-images/` - Custom backgrounds
- `/lxcdata/planka/attachments/` - Uploaded files

**Database dump:**
```bash
pct exec 122 -- bash -c 'docker exec postgres pg_dumpall -U planka' > planka-backup.sql
```

**Restore database:**
```bash
cat planka-backup.sql | pct exec 122 -- bash -c 'docker exec -i postgres psql -U planka'
```

**Full backup:**
```bash
rsync -av /lxcdata/planka/ /backup/homelab/planka/
```

## Maintenance

**Update:**
```bash
pct exec 122 -- bash -c 'cd /opt/homelab-docker/planka && docker compose pull && docker compose up -d'
```

**View logs:**
```bash
pct exec 122 -- bash -c 'cd /opt/homelab-docker/planka && docker compose logs -f'
```

**Restart:**
```bash
pct exec 122 -- bash -c 'cd /opt/homelab-docker/planka && docker compose restart'
```

## Common Tasks

**Create project:** Via web UI
**Invite users:** Via web UI (share link or invite)
**Customize boards:** Via web UI

## Related

- [[infrastructure/network|Network routing]]
- [[how-to/update-service|Update a Service]]
- [[how-to/backup-restore|Backup & Restore]]
