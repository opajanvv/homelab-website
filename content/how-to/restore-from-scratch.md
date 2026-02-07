---
type: how-to
tags: [disaster-recovery, migration, rebuild]
last-tested: 2026-02-06
---

# How to Restore from Scratch

Complete disaster recovery - rebuild homelab from nothing.

## When to use

- Complete Proxmox host failure
- Migrating to new hardware
- Catastrophic data loss

## Prerequisites

- [ ] Proxmox VE installed on new hardware
- [ ] Network configured (same subnet: 192.168.144.0/23)
- [ ] Backup of `/lxcdata/` (service data)
- [ ] Backup of LXC/VM configs
- [ ] Copy of `~/dev/homelab-docker/` repository

## Steps

### 1. Prepare Proxmox host

Install Proxmox VE and configure:
- **Network:** 192.168.144.0/23, gateway .1
- **Storage:** ZFS for `/lxcdata` and VM storage
- **Network bridge:** vmbr0

### 2. Create base template (CT 902)

```bash
# Create Arch Linux LXC
# (Download Arch Linux template in Proxmox GUI first)
pct create 902 local:vztmpl/archlinux-lxc-template_*.tar.zst \
  --hostname lxc-base \
  --cores 2 \
  --memory 2048 \
  --net0 name=eth0,bridge=vmbr0,ip=dhcp \
  --features nesting=1,keyctl=1

# Start template
pct start 902

# Disable sandbox in pacman.conf
pct exec 902 -- sed -i '/\[options\]/a DisableSandbox' /etc/pacman.conf

# Install Docker
pct exec 902 -- pacman -Sy docker docker-compose
pct exec 902 -- systemctl enable docker

# Stop template
pct stop 902
```

### 3. Restore service data

```bash
# Copy service data from backup
rsync -av /backup/homelab/ /lxcdata/
```

### 4. Recreate containers

For each service:

```bash
# Clone template
pct clone 902 <CT_ID> --hostname <service> --full

# Set resources
pct set <CT_ID> --cores <cores> --memory <ram>

# Configure network
pct set <CT_ID> -net0 name=eth0,bridge=vmbr0,firewall=1,gw=192.168.144.1,ip=<IP>/23

# Mount data directory
pct set <CT_ID> -mp0 /lxcdata/<service>,mp=/data

# Enable features
pct set <CT_ID> -features nesting=1,keyctl=1

# Auto-start
pct set <CT_ID> -onboot 1

# Add AppArmor workaround
cat >> /etc/pve/lxc/<CT_ID>.conf << 'EOF'
lxc.apparmor.profile: unconfined
lxc.mount.entry: /dev/null sys/module/apparmor/parameters/enabled none bind 0 0
EOF

# Start and deploy
pct start <CT_ID>
pct exec <CT_ID> -- systemctl start docker
pct exec <CT_ID> -- bash -c 'git clone https://github.com/opajanvv/homelab-docker.git /opt/homelab-docker'
pct exec <CT_ID> -- bash -c 'cd /opt/homelab-docker/<service> && chmod +x install.sh && ./install.sh'
```

### 5. Recreate VMs (Immich, Home Assistant)

For each VM:
- Create VM with appropriate resources
- Restore VM disk from backup or reinstall OS
- Configure networking

### 6. Restore databases

**MariaDB (WordPress):**
```bash
# After wordpress-db container is running
cat mariadb-backup.sql | pct exec 128 -- bash -c 'docker exec -i mariadb mysql -u root -p'
```

**PostgreSQL (Planka):**
```bash
cat planka-backup.sql | pct exec 122 -- bash -c 'docker exec -i postgres psql -U planka'
```

### 7. Recreate routing

**Cloudflare Tunnel:**
- Log into Cloudflare Zero Trust dashboard
- Recreate tunnel or reconfigure existing tunnel
- Add all routes (see [[../infrastructure/network|network documentation]])

**Lanproxy (Caddy):**
- Reinstall lanproxy container
- Restore Caddyfile from backup
- Reload configuration

### 8. Verify all services

Go through each service:
- [ ] Container running
- [ ] Service responding
- [ ] External access working
- [ ] Internal access working

## Service Priority Order

Restore in this order:

1. **Infrastructure:** wordpress-db, lanproxy, cloudflared
2. **Critical services:** n8n, planka
3. **Web services:** grav, wordpress sites
4. **Other services:** jellyfin, ai, syncthing, immich, home-assistant

## Estimated Times

- **Proxmox setup:** 1-2 hours
- **Template creation:** 30 minutes
- **Per service:** 10-15 minutes
- **Total:** 4-6 hours for full restore

## Verification

- [ ] All containers running: `pct list` shows "running"
- [ ] All services accessible via web
- [ ] External routes working
- [ ] Data intact (check a few key services)
- [ ] Backups working

## Related

- [[./backup-restore|Backup & Restore]]
- [[../infrastructure/proxmox|Proxmox Configuration]]
- [[./deploy-new-service|Deploy New Service]]
