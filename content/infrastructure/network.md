---
type: infrastructure
updated: 2026-02-06
---

# Network Topology

The homelab network provides internal communication, external access via tunnels, and DNS-based service discovery.

## Network Configuration

**Subnet:** 192.168.144.0/23 (512 addresses, 192.168.144.0 - 192.168.144.255)
**Gateway:** 192.168.144.1
**DNS:** 192.168.144.20 (Pi-hole)

## IP Allocation

### Infrastructure

| Device | IP | Role |
|--------|-----|------|
| Gateway | 192.168.144.1 | Router |
| Pi-hole | 192.168.144.20 | DNS + ad blocking |
| Proxmox | 192.168.144.10 | Virtualization host |
| Uptime Kuma | 192.168.144.25 | Monitoring |

### Services by Category

**Networking:**
- Lanproxy (Caddy): 192.168.144.31
- Cloudflared: 192.168.144.32

**Databases:**
- WordPress (MariaDB): 192.168.144.41

**Applications:**
- Planka: 192.168.144.60
- n8n: 192.168.144.61
- AI (Ollama): 192.168.144.62
- Syncthing: 192.168.144.63

**Web/CMS:**
- WordPress jokegoudriaan: 192.168.144.70
- WordPress kledingruil: 192.168.144.71
- Grav: 192.168.144.72
- WordPress pgh: 192.168.144.73

**Media:**
- Jellyfin: 192.168.144.100

**AI/Media:**
- Immich (VM): 192.168.144.110
- Home Assistant (VM): 192.168.144.120

## DNS

### Pi-hole (192.168.144.20)

- **Purpose:** Network-wide ad blocking + DNS resolution
- **Web UI:** https://pihole.janvv.nl
- **Local DNS:** Resolves `.janvv.nl` domains to internal IPs

### Split-Horizon DNS

External clients use Cloudflare DNS → Cloudflare Tunnel
Internal clients use Pi-hole → Local IP → Lanproxy (Caddy) → Service

This enables:
- Same URL works internally and externally
- Internal traffic stays local (faster, no tunnel overhead)
- Automatic SSL via Cloudflare DNS-01 challenge

## Routing

### External Access (Cloudflare Tunnel - CT 126)

Routes external traffic through Cloudflare's network to internal services.

**Configured routes:**
| External URL | Internal Destination |
|--------------|---------------------|
| https://tasks.janvv.nl | 192.168.144.60:1337 (Planka) |
| https://n8n.janvv.nl | 192.168.144.61:5678 (n8n) |
| https://opa.janvv.nl | 192.168.144.72:80 (Grav) |
| https://kijkdoos.janvv.nl | 192.168.144.100:8096 (Jellyfin) |
| https://jokegoudriaan.nl | 192.168.144.70:80 (WordPress) |
| https://kledingruil.jokegoudriaan.nl | 192.168.144.71:80 (WordPress) |
| https://pgh.janvv.nl | 192.168.144.73:80 (WordPress) |
| https://assistant.janvv.nl | 192.168.144.120:8123 (Home Assistant) |
| https://photos.janvv.nl | 192.168.144.110:2283 (Immich) |
| https://pihole.janvv.nl | 192.168.144.20:80 (Pi-hole) |
| https://proxmox.janvv.nl | 192.168.144.10:8006 (Proxmox) |
| https://status.janvv.nl | 192.168.144.25:80 (Uptime Kuma) |
| https://sync.janvv.nl | 192.168.144.63:8384 (Syncthing) |

**Management:** Cloudflare Zero Trust dashboard (Zero Trust → Networks → Tunnels)

### Internal HTTPS (Lanproxy/Caddy - CT 127)

Caddy reverse proxy provides internal HTTPS with automatic certificates.

**Configuration:** `/opt/homelab-docker/lanproxy/Caddyfile` in CT 127

**Benefits:**
- HTTPS on local network
- Automatic SSL via Cloudflare DNS-01 challenge
- Centralized reverse proxy configuration

## Related

- [[services/cloudflared|Cloudflared Service]]
- [[services/lanproxy|Lanproxy (Caddy) Service]]
- [[how-to/deploy-new-service|Adding New Routes]]
