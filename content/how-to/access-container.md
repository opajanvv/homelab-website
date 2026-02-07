---
type: how-to
tags: [access, ssh, containers, commands]
last-tested: 2026-02-06
---

# How to Access Containers

Methods for accessing LXC containers and running commands.

## When to use

- Checking service status
- Running maintenance commands
- Debugging issues
- Viewing logs

## Prerequisites

- [ ] Access to Proxmox host
- [ ] Know the CT ID of the container

## Methods

### 1. Execute single command (from Proxmox host)

```bash
# Run command in container
pct exec <CT_ID> -- <command>

# Examples
pct exec 120 -- docker ps
pct exec 120 -- ls -la /data
pct exec 120 -- systemctl status docker
```

### 2. Interactive shell (from Proxmox host)

```bash
# Access container shell
pct exec <CT_ID> -- bash

# Or using enter
pct enter <CT_ID>
```

### 3. SSH directly (if SSH server installed)

```bash
# SSH to container
ssh root@<container-ip>

# Example
ssh root@192.168.144.61
```

## Common Commands

### Container management

```bash
# List all containers
pct list

# Start/Stop/Restart
pct start <CT_ID>
pct stop <CT_ID>
pct restart <CT_ID>

# View container config
cat /etc/pve/lxc/<CT_ID>.conf

# View container status
pct status <CT_ID>
```

### Docker commands (inside container)

```bash
# List running containers
docker ps

# List all containers (including stopped)
docker ps -a

# View logs
docker logs <container-name>

# Execute command in container
docker exec -it <container-name> bash
```

### Docker Compose commands

```bash
# Navigate to service directory
cd /opt/homelab-docker/<service>

# View status
docker compose ps

# View logs
docker compose logs -f

# Restart service
docker compose restart

# Stop service
docker compose down

# Start service
docker compose up -d

# Pull new images
docker compose pull
```

## Quick Reference: Common CT IDs

| CT ID | Service |
|-------|---------|
| 120 | n8n |
| 121 | Syncthing |
| 122 | Planka |
| 123 | Grav |
| 124 | AI Services |
| 125 | Jellyfin |
| 126 | Cloudflared |
| 127 | Lanproxy |
| 128 | WordPress DB |
| 129 | WordPress jokegoudriaan |
| 130 | WordPress kledingruil |
| 131 | WordPress pgh |
| 108 | Immich (VM) |
| 114 | Home Assistant (VM) |

## Quick Reference: Common IP Addresses

| IP | Service |
|-----|---------|
| 192.168.144.31 | Lanproxy |
| 192.168.144.32 | Cloudflared |
| 192.168.144.41 | WordPress DB |
| 192.168.144.60 | Planka |
| 192.168.144.61 | n8n |
| 192.168.144.62 | AI Services |
| 192.168.144.63 | Syncthing |
| 192.168.144.70 | WordPress jokegoudriaan |
| 192.168.144.71 | WordPress kledingruil |
| 192.168.144.72 | Grav |
| 192.168.144.73 | WordPress pgh |
| 192.168.144.100 | Jellyfin |
| 192.168.144.110 | Immich (VM) |
| 192.168.144.120 | Home Assistant (VM) |

## Related

- [[../infrastructure/proxmox|Proxmox Basics]]
- [[./troubleshoot-service|Troubleshooting Guide]]
