---
service: Immich
status: running
host: 192.168.144.110
vm: 108
port: 2283
tags: [app, photos, backup]
updated: 2026-02-07
---

# Immich

Self-hosted photo and video backup solution.

## Overview

Immich provides high-performance photo management with backup, mobile apps, and AI features. Alternative to Google Photos.

**Access:** https://photos.janvv.nl
**VM:** 108
**IP:** 192.168.144.110
**Port:** 2283
**Resources:** 2 cores, 4GB RAM, 96GB disk

## Deployment

Immich runs on a dedicated VM, not an LXC container.

```bash
# Create VM
qm create 108 --name immich --cores 2 --memory 4096 --net0 virtio,bridge=vmbr0
qm set 108 --ciuser jan
# ... additional VM configuration

# Install Immich (VM-specific)
# Follow Immich installation guide for the VM's OS
```

## Configuration

Immich consists of multiple services:
- **Web** - Frontend application
- **Server** - API backend
- **Microservices** - Background processing (ML, video transcoding)
- **PostgreSQL** - Database
- **Redis** - Cache
- **TypeSense** - Search

**Storage:**
- **96GB VM disk** - Contains all Immich data (photos, database, configs)
- Photos are stored on the VM's local disk, not in `/vmdata/`
- An empty `/vmdata/immich-photos/` directory exists but is unused

## Access

- **Web UI:** https://photos.janvv.nl
- **Direct:** http://192.168.144.110:2283
- **Mobile Apps:** Available for iOS and Android

**First Setup:**
1. Create admin account
2. Set up library paths
3. Install mobile app and connect to server

## Backup

**What to backup:**
- **VM snapshot** - Easiest: snapshot the entire VM
- **Database:** PostgreSQL dump (from within VM)
- **Photos:** Stored on VM disk

**Note:** Immich primarily serves as a backup/management tool for photos. Consider off-site backup for the VM disk or photo library.

## Maintenance

**Update:** Follow Immich update documentation (docker-compose pull or git pull depending on installation)

**Logs:** Check VM logs for Immich services

## Common Tasks

**Upload photos:** Mobile app or web upload
**Create album:** Web UI or mobile app
**Share album:** Generate share link from album
**AI search:** Natural language search for photos (built-in)

## Related

- [[infrastructure/network|Network routing]]
- [[services/jellyfin|Jellyfin (media server)]]
- [[infrastructure/storage|Storage Configuration]]
