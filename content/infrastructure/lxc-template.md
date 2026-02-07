---
type: infrastructure
updated: 2026-02-06
---

# LXC Base Template

CT 902 is the Arch Linux template used as the foundation for all Docker-based LXC containers.

## Template Overview

**CT ID:** 902
**Hostname:** lxc-base
**OS:** Arch Linux
**Tag:** Template
**Purpose:** Base for cloning new service containers

## Requirements

The template must have:

### 1. Pacman Configuration

Edit `/etc/pacman.conf` and add under `[options]`:
```
DisableSandbox
```

**Why:** Fixes Landlock kernel issue in unprivileged LXC containers (pacman can't create sandboxed process).

### 2. Docker Installation

```bash
pacman -S docker docker-compose
systemctl enable docker
# Do NOT start docker (start after cloning)
```

### 3. User Configuration

No special user configuration needed - containers run as root, Docker handles user mapping.

## Using the Template

### Cloning

```bash
# Clone to new container
pct clone 902 <NEW_ID> --hostname <service-name> --full

# Set resources
pct set <NEW_ID> --cores 2 --memory 2048

# Configure network
pct set <NEW_ID> -net0 name=eth0,bridge=vmbr0,firewall=1,gw=192.168.144.1,ip=<IP>/23

# Mount data directory
pct set <NEW_ID> -mp0 /lxcdata/<service>,mp=/data

# Enable features
pct set <NEW_ID> -features nesting=1,keyctl=1

# Auto-start on boot
pct set <NEW_ID> -onboot 1
```

### Post-Clone Setup

```bash
# Start container
pct start <NEW_ID>

# Start Docker (inside container)
pct exec <NEW_ID> -- systemctl start docker

# Verify
pct exec <NEW_ID> -- docker ps
```

### Deploying a Service

```bash
# Clone homelab-docker repo
pct exec <NEW_ID> -- bash -c 'git clone https://github.com/opajanvv/homelab-docker.git /opt/homelab-docker'

# Deploy service
pct exec <NEW_ID> -- bash -c 'cd /opt/homelab-docker/<service> && chmod +x install.sh && ./install.sh'
```

## Maintaining the Template

Periodically update the template to keep base packages current:

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

**Note:** After updating, consider creating new services from the updated template. Existing containers are not automatically updated.

## Template Verification

To verify a container was created from the template:

```bash
# Check for Docker
pct exec <CT_ID> -- which docker

# Check docker-compose
pct exec <CT_ID> -- which docker-compose

# Check DisableSandbox in pacman.conf
pct exec <CT_ID> -- grep DisableSandbox /etc/pacman.conf
```

## Related

- [[infrastructure/proxmox|Proxmox Basics]]
- [[how-to/deploy-new-service|Deploy a New Service]]
