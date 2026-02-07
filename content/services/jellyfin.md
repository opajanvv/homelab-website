---
service: Jellyfin
status: running
host: 192.168.144.100
ct: 125
port: 8096
tags: [media, streaming]
updated: 2026-02-06
---

# Jellyfin

Media server for streaming movies, TV shows, and music.

## Overview

Jellyfin is a free software media system that provides streaming media from a dedicated server. Replaces Plex.

**Access:** https://kijkdoos.janvv.nl
**Container:** CT 125
**IP:** 192.168.144.100
**Port:** 8096

## Source Code

**Docker Compose & Install Script:** [github.com/opajanvv/homelab-docker/tree/main/jellyfin](https://github.com/opajanvv/homelab-docker/tree/main/jellyfin)
**Local copy:** `~/dev/homelab-docker/jellyfin/`

## Deployment

```bash
# Clone LXC template
pct clone 902 125 --hostname jellyfin --full
pct set 125 --cores 2 --memory 2048
pct set 125 -net0 name=eth0,bridge=vmbr0,firewall=1,gw=192.168.144.1,ip=192.168.144.100/23
pct set 125 -mp0 /lxcdata/jellyfin,mp=/data
pct set 125 -features nesting=1,keyctl=1
pct set 125 -onboot 1

# Add AppArmor workaround
cat >> /etc/pve/lxc/125.conf << 'EOF'
lxc.apparmor.profile: unconfined
lxc.mount.entry: /dev/null sys/module/apparmor/parameters/enabled none bind 0 0
EOF

# Deploy
pct start 125
pct exec 125 -- bash -c 'systemctl enable --now docker'
pct exec 125 -- bash -c 'git clone https://github.com/opajanvv/homelab-docker.git /opt/homelab-docker'
pct exec 125 -- bash -c 'cd /opt/homelab-docker/jellyfin && chmod +x install.sh && ./install.sh'
```

## Configuration

**Data Locations:**
- `/data/config/` - Jellyfin configuration
- `/data/media/` - Media library (movies, TV, music)
- `/opt/homelab-docker/jellyfin/` - Docker Compose config (inside container)
- `~/dev/homelab-docker/jellyfin/` - Local working directory (laptop)

**Media Setup:**
Media files should be mounted or stored in `/data/media/`. Organize as:
```
/data/media/
├── movies/
├── tvshows/
└── music/
```

## Access

- **Web UI:** https://kijkdoos.janvv.nl
- **Direct:** http://192.168.144.100:8096

**First Setup:** Create admin account, add media libraries

## Backup

**What to backup:**
- `/lxcdata/jellyfin/config/` - Configuration, user data, metadata
- `/lxcdata/jellyfin/media/` - Media files (if not backed up elsewhere)

**Backup command:**
```bash
rsync -av /lxcdata/jellyfin/config/ /backup/homelab/jellyfin-config/
```

**Media backup:** Depends on your media storage strategy. If media is on separate storage, back up separately.

## Maintenance

**Update:**
```bash
pct exec 125 -- bash -c 'cd /opt/homelab-docker/jellyfin && docker compose pull && docker compose up -d'
```

**View logs:**
```bash
pct exec 125 -- bash -c 'cd /opt/homelab-docker/jellyfin && docker compose logs -f'
```

**Restart:**
```bash
pct exec 125 -- bash -c 'cd /opt/homelab-docker/jellyfin && docker compose restart'
```

## Common Tasks

**Add media library:** Web UI → Dashboard → Media → Library → Add Media Library
**Scan for new files:** Web UI → Dashboard → Media Library → Scan
**Transcode settings:** Web UI → Dashboard → Playback → Transcoding

## Related

- [[infrastructure/network|Network routing]]
- [[how-to/update-service|Update a Service]]
- [[how-to/backup-restore|Backup & Restore]]
