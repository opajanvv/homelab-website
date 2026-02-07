---
service: Lanproxy
status: running
host: 192.168.144.31
ct: 127
port: 80/443
tags: [network, reverse-proxy, internal-https]
updated: 2026-02-06
---

# Lanproxy (Caddy Reverse Proxy)

Internal HTTPS reverse proxy with automatic SSL certificates.

## Overview

Caddy provides HTTPS access to all services on the local network using Cloudflare DNS-01 challenge for automatic SSL certificates. Works with Pi-hole's split-horizon DNS.

**Container:** CT 127
**IP:** 192.168.144.31
**Ports:** 80 (HTTP), 443 (HTTPS)

## Source Code

**Docker Compose & Install Script:** [github.com/opajanvv/homelab-docker/tree/main/lanproxy](https://github.com/opajanvv/homelab-docker/tree/main/lanproxy)
**Local copy:** `~/dev/homelab-docker/lanproxy/`

## Deployment

```bash
# Clone LXC template
pct clone 902 127 --hostname lanproxy --full
pct set 127 --cores 1 --memory 512
pct set 127 -net0 name=eth0,bridge=vmbr0,firewall=1,gw=192.168.144.1,ip=192.168.144.31/23
pct set 127 -mp0 /lxcdata/lanproxy,mp=/data
pct set 127 -features nesting=1,keyctl=1
pct set 127 -onboot 1

# Deploy
pct start 127
pct exec 127 -- bash -c 'systemctl enable --now docker'
pct exec 127 -- bash -c 'git clone https://github.com/opajanvv/homelab-docker.git /opt/homelab-docker'
pct exec 127 -- bash -c 'cd /opt/homelab-docker/lanproxy && chmod +x install.sh && ./install.sh'
```

## Configuration

**Environment Variable:**
- `CLOUDFLARE_API_TOKEN` - Cloudflare API token for DNS-01 challenge

**Data Locations:**
- `/data/` - Caddy data (certificates, logs)
- `/opt/homelab-docker/lanproxy/` - Docker Compose config, Caddyfile

**Config File:** `/opt/homelab-docker/lanproxy/Caddyfile`

**Caddyfile Format:**
```
<service-domain> {
    reverse_proxy <internal-ip>:<port>
}
```

**Example:**
```
tasks.janvv.nl {
    reverse_proxy 192.168.144.60:1337
}

n8n.janvv.nl {
    reverse_proxy 192.168.144.61:5678
}
```

## How It Works

1. **Internal client** requests `https://tasks.janvv.nl`
2. **Pi-hole DNS** (split-horizon) resolves `tasks.janvv.nl` to `192.168.144.31` (Lanproxy)
3. **Caddy** receives request, gets SSL cert via Cloudflare DNS-01
4. **Caddy** proxies to `192.168.144.60:1337` (Planka)
5. **Response** flows back through Caddy to client

## Backup

**What to backup:**
- `/opt/homelab-docker/lanproxy/Caddyfile` - Route configuration
- `/lxcdata/lanproxy/` - Caddy data (certificates, logs)

**Backup command:**
```bash
cp /opt/homelab-docker/lanproxy/Caddyfile /backup/homelab/
rsync -av /lxcdata/lanproxy/ /backup/homelab/lanproxy/
```

## Maintenance

**Update:**
```bash
pct exec 127 -- bash -c 'cd /opt/homelab-docker/lanproxy && docker compose pull && docker compose up -d'
```

**View logs:**
```bash
pct exec 127 -- bash -c 'cd /opt/homelab-docker/lanproxy && docker compose logs -f'
```

**Reload config after Caddyfile change:**
```bash
pct exec 127 -- bash -c 'cd /opt/homelab-docker/lanproxy && docker compose restart'
```

## Adding a New Route

1. Edit Caddyfile:
   ```bash
   pct exec 127 -- nano /opt/homelab-docker/lanproxy/Caddyfile
   ```

2. Add new route:
   ```
   new-service.janvv.nl {
       reverse_proxy 192.168.144.XX:PORT
   }
   ```

3. Reload Caddy:
   ```bash
   pct exec 127 -- bash -c 'cd /opt/homelab-docker/lanproxy && docker compose restart'
   ```

4. Update Pi-hole DNS to point `new-service.janvv.nl` to `192.168.144.31`

## Troubleshooting

**Certificate errors:** Check Cloudflare API token is valid
**Route not working:** Verify service is accessible from Lanproxy container
**DNS issues:** Check Pi-hole DNS configuration

## Related

- [[services/cloudflared|Cloudflared (External Access)]]
- [[infrastructure/network|Network Topology]]
