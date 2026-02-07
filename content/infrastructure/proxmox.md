---
type: infrastructure
updated: 2026-02-06
---

# Proxmox VE

Proxmox Virtual Environment is the foundation of the homelab. It hosts all LXC containers and VMs.

## Overview

**Purpose:** Virtualization host for all homelab services
**Access:** https://proxmox.janvv.nl (via Cloudflare tunnel)
**IP:** 192.168.144.10

## Container vs VM

**LXC Containers (preferred):**
- Lightweight, share host kernel
- Used for Docker-based services
- Template: CT 902 (Arch Linux lxc-base)
- Lower resource overhead

**Virtual Machines:**
- Full OS isolation
- Used when: needs direct hardware access, or OS not supported by LXC
- Immich (VM 108), Home Assistant (VM 114)

## LXC Basics

### Template Container

**CT 902 - lxc-base**
- OS: Arch Linux
- Docker and docker-compose pre-installed
- Docker service enabled (not started)
- `DisableSandbox` in `/etc/pacman.conf` (fixes Landlock kernel issue)

### Container Creation

```bash
# Clone template
pct clone 902 <NEW_ID> --hostname <name> --full

# Set resources
pct set <NEW_ID> --cores 2 --memory 2048

# Configure network
pct set <NEW_ID> -net0 name=eth0,bridge=vmbr0,firewall=1,gw=192.168.144.1,ip=<IP>/23

# Mount data directory
pct set <NEW_ID> -mp0 /lxcdata/<service>,mp=/data

# Enable features
pct set <NEW_ID> -features nesting=1,keyctl=1

# Auto-start
pct set <NEW_ID> -onboot 1

# Create data directory on host
mkdir -p /lxcdata/<service>
```

### AppArmor Workaround

Due to CVE-2025-52881, Docker-in-LXC requires AppArmor workaround. Add to `/etc/pve/lxc/<ID>.conf`:

```
lxc.apparmor.profile: unconfined
lxc.mount.entry: /dev/null sys/module/apparmor/parameters/enabled none bind 0 0
```

### Common Commands

```bash
# List all containers
pct list

# Start/Stop/Restart
pct start <ID>
pct stop <ID>
pct restart <ID>

# Access shell
pct exec <ID> -- bash

# Run command
pct exec <ID> -- <command>

# View config
cat /etc/pve/lxc/<ID>.conf

# View status
pct status <ID>

# Delete container
pct destroy <ID>
```

## VM Basics

### VM Creation

Similar to LXC but using `qm` commands:
```bash
qm create <VMID> --name <name> --cores 2 --memory 4096
qm set <VMID> --net0 virtio,bridge=vmbr0
# ... additional config
```

### Common Commands

```bash
qm list                    # List VMs
qm start <ID>              # Start VM
qm stop <ID>               # Stop VM
qm status <ID>             # View status
```

## Related

- [[storage|ZFS Storage Configuration]]
- [[network|Network Topology]]
- [[lxc-template|LXC Template Requirements]]
- [[how-to/deploy-new-service|Deploy a New Service]]
