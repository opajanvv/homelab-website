---
service: Cloudflared
status: running
host: 192.168.144.32
ct: 126
tags: [network, tunnel, external-access]
updated: 2026-02-06
---

# Cloudflared (Cloudflare Tunnel)

Provides external access to homelab services without opening ports.

## Overview

Cloudflare Tunnel creates a secure outbound connection from the homelab to Cloudflare's edge. External traffic comes through Cloudflare, then through the tunnel to internal services.

**Container:** CT 126
**IP:** 192.168.144.32
**Purpose:** External access routing

## Source Code

**Docker Compose & Install Script:** [github.com/opajanvv/homelab-docker/tree/main/tunnel](https://github.com/opajanvv/homelab-docker/tree/main/tunnel)
**Local copy:** `~/dev/homelab-docker/tunnel/`

## Deployment

```bash
# Clone LXC template
pct clone 902 126 --hostname cloudflared --full
pct set 126 --cores 1 --memory 512
pct set 126 -net0 name=eth0,bridge=vmbr0,firewall=1,gw=192.168.144.1,ip=192.168.144.32/23
pct set 126 -features nesting=1,keyctl=1
pct set 126 -onboot 1

# No data mount needed for tunnel

# Deploy
pct start 126
pct exec 126 -- bash -c 'systemctl enable --now docker'
pct exec 126 -- bash -c 'git clone https://github.com/opajanvv/homelab-docker.git /opt/homelab-docker'
pct exec 126 -- bash -c 'cd /opt/homelab-docker/tunnel && chmod +x install.sh && ./install.sh'
```

## Configuration

**Environment Variable:**
- `TUNNEL_TOKEN` - Cloudflare tunnel token (from Cloudflare dashboard)

**Data Location:**
- No persistent data needed
- Config stored in Cloudflare dashboard

## Routing Configuration

Routes are configured in **Cloudflare Zero Trust dashboard**:

1. Go to: Zero Trust → Networks → Tunnels
2. Select your tunnel
3. Add "Public Hostname" for each service

**Current Routes:**
| External URL | Internal Destination |
|--------------|---------------------|
| https://tasks.janvv.nl | 192.168.144.60:1337 |
| https://n8n.janvv.nl | 192.168.144.61:5678 |
| https://opa.janvv.nl | 192.168.144.72:80 |
| https://kijkdoos.janvv.nl | 192.168.144.100:8096 |
| https://jokegoudriaan.nl | 192.168.144.70:80 |
| https://kledingruil.jokegoudriaan.nl | 192.168.144.71:80 |
| https://pgh.janvv.nl | 192.168.144.73:80 |
| https://assistant.janvv.nl | 192.168.144.120:8123 |
| https://photos.janvv.nl | 192.168.144.110:2283 |
| https://pihole.janvv.nl | 192.168.144.20:80 |
| https://proxmox.janvv.nl | 192.168.144.10:8006 |
| https://status.janvv.nl | 192.168.144.25:80 |
| https://sync.janvv.nl | 192.168.144.63:8384 |

## Backup

No persistent data to backup. Tunnel routes are stored in Cloudflare.

**Document:** Keep a list of configured routes (as above) for reference.

## Maintenance

**Update:**
```bash
pct exec 126 -- bash -c 'cd /opt/homelab-docker/tunnel && docker compose pull && docker compose up -d'
```

**View logs:**
```bash
pct exec 126 -- bash -c 'cd /opt/homelab-docker/tunnel && docker compose logs -f'
```

**Check tunnel status:** Cloudflare Zero Trust dashboard → Tunnels → Your tunnel

## Adding a New Route

1. Go to Cloudflare Zero Trust → Networks → Tunnels
2. Select your tunnel
3. Click "Public Hostname"
4. Add:
   - Subdomain/domain
   - Service type: HTTP
   - URL: `http://<internal-ip>:<port>`
5. Save

## Troubleshooting

**Tunnel offline:** Check container is running, check token is valid
**Route not working:** Verify internal service is accessible from tunnel container

## Related

- [[services/lanproxy|Lanproxy (Internal HTTPS)]]
- [[infrastructure/network|Network Topology]]
- [[how-to/deploy-new-service|Deploy New Service]]
