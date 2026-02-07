---
type: how-to
tags: [deployment, containers, services]
last-tested: 2026-02-06
---

# How to Deploy a New Service

End-to-end guide for adding a new service to the homelab.

## When to use

Adding a new Docker-based service that will run in its own LXC container.

## Prerequisites

- [ ] Proxmox host access
- [ ] CT 902 (lxc-base template) exists and is configured
- [ ] Service has a Docker Compose configuration
- [ ] Know the service's resource requirements (CPU, RAM)
- [ ] Have an IP address available (check [[../infrastructure/network|network topology]])

## Steps

### 1. Create the service configuration

In `~/dev/homelab-docker/` on your local machine:

```bash
mkdir <service>
cd <service>
```

Create `docker-compose.yml`:
```yaml
version: "3"
services:
  app:
    image: image-name:latest
    container_name: <service>
    restart: unless-stopped
    volumes:
      - /data/<subdir>:/path/in/container
    ports:
      - "port:port"
    environment:
      - VAR=value
```

Create `.env.example` with environment variable templates (no secrets).

Create `install.sh`:
```bash
#!/bin/bash
set -e

# Validate required variables
if [ ! -f .env ]; then
    echo "Error: .env file not found"
    echo "Create .env from .env.example"
    exit 1
fi

# Create data directories
mkdir -p /data/<subdirs>

# Start service
docker compose up -d
```

Create `README.md` documenting the service (CT, IP, ports, access).

### 2. Create LXC container

On Proxmox host:

```bash
# Clone template
pct clone 902 <NEW_ID> --hostname <service> --full

# Set resources (adjust as needed)
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

### 3. Add AppArmor workaround

Required for Docker in LXC (CVE-2025-52881):

```bash
cat >> /etc/pve/lxc/<NEW_ID>.conf << 'EOF'
lxc.apparmor.profile: unconfined
lxc.mount.entry: /dev/null sys/module/apparmor/parameters/enabled none bind 0 0
EOF
```

### 4. Create data directory

```bash
mkdir -p /lxcdata/<service>
```

### 5. Start and deploy

```bash
# Start container
pct start <NEW_ID>

# Start Docker
pct exec <NEW_ID> -- bash -c 'systemctl enable --now docker'

# Clone homelab-docker
pct exec <NEW_ID> -- bash -c 'git clone https://github.com/opajanvv/homelab-docker.git /opt/homelab-docker'

# Create .env file
pct exec <NEW_ID> -- bash -c 'cp /opt/homelab-docker/<service>/.env.example /opt/homelab-docker/<service>/.env'
pct exec <NEW_ID> -- nano /opt/homelab-docker/<service>/.env
# Add actual values to .env

# Deploy service
pct exec <NEW_ID> -- bash -c 'cd /opt/homelab-docker/<service> && chmod +x install.sh && ./install.sh'
```

### 6. Configure external access (if needed)

**For external access:** Add route in Cloudflare Zero Trust dashboard

**For internal HTTPS:** Add to Lanproxy Caddyfile:
```
your-service.janvv.nl {
    reverse_proxy <service-ip>:<port>
}
```

Then reload Lanproxy:
```bash
pct exec 127 -- bash -c 'cd /opt/homelab-docker/lanproxy && docker compose restart'
```

## Verification

- [ ] Container is running: `pct status <NEW_ID>` shows "running"
- [ ] Docker containers running: `pct exec <NEW_ID> -- docker ps`
- [ ] Service accessible: `curl http://<service-ip>:<port>`
- [ ] External route working (if configured)

## Troubleshooting

**Container won't start:** Check [[./troubleshoot-service|troubleshooting guide]]
**Docker issues:** Verify AppArmor workaround is applied
**Network issues:** Check firewall rules and IP availability

## Related

- [[../infrastructure/lxc-template|LXC Template Requirements]]
- [[../infrastructure/proxmox|Proxmox Basics]]
- [[./access-container|Access Containers Guide]]
