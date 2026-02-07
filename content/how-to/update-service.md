---
type: how-to
tags: [maintenance, updates, docker]
last-tested: 2026-02-06
---

# How to Update a Service

Update Docker images and containers for a service.

## When to use

- Regular maintenance
- Security updates available
- New features needed

## Prerequisites

- [ ] Know the service container CT ID
- [ ] Service is currently running

## Steps

### 1. Check current status

```bash
# Check container is running
pct status <CT_ID>

# Check running Docker containers
pct exec <CT_ID> -- docker ps
```

### 2. Pull new images

```bash
pct exec <CT_ID> -- bash -c 'cd /opt/homelab-docker/<service> && docker compose pull'
```

### 3. Restart with new images

```bash
pct exec <CT_ID> -- bash -c 'cd /opt/homelab-docker/<service> && docker compose up -d'
```

This will:
- Pull new images
- Stop containers with old images
- Start containers with new images
- Preserve data volumes

### 4. Verify service is working

```bash
# Check container status
pct exec <CT_ID> -- docker ps

# Check logs
pct exec <CT_ID> -- bash -c 'cd /opt/homelab-docker/<service> && docker compose logs -f'

# Test service
curl http://<service-ip>:<port>
```

## Updating Multiple Services

```bash
# List services to update
for ct in 120 121 122 123 124 125; do
    echo "Updating CT $ct..."
    pct exec $ct -- bash -c 'cd /opt/homelab-docker && git pull'
done

# Then update each service individually
```

## Updating the LXC Template

Periodically update CT 902 (lxc-base):

```bash
# Start template
pct start 902

# Update system
pct exec 902 -- pacman -Syu

# Update Docker
pct exec 902 -- pacman -S docker docker-compose

# Stop template
pct stop 902
```

New services cloned after this will have updated packages. Existing containers are not affected.

## Rollback

If an update breaks something:

### Option 1: Restart with previous image

```bash
# Find previous image
pct exec <CT_ID> -- docker images | grep <service>

# Edit docker-compose.yml to use previous image tag
pct exec <CT_ID> -- nano /opt/homelab-docker/<service>/docker-compose.yml

# Restart
pct exec <CT_ID> -- bash -c 'cd /opt/homelab-docker/<service> && docker compose up -d'
```

### Option 2: Restore from backup

See [[./backup-restore|Backup & Restore]]

## Verification

- [ ] Service container is running
- [ ] Service responds to requests
- [ ] No errors in logs
- [ ] Data is intact

## Troubleshooting

**Container won't start after update:**
```bash
# Check logs
pct exec <CT_ID> -- bash -c 'cd /opt/homelab-docker/<service> && docker compose logs'

# Rollback to previous image version
```

**Image pull fails:**
- Check network connectivity
- Check image name is correct
- Try `docker pull <image>` manually

## Related

- [[./access-container|Access Containers]]
- [[./backup-restore|Backup & Restore]]
