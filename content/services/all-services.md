---
type: reference
updated: 2026-02-07
---

# All Services

Complete inventory of homelab services with quick reference information.

## Quick Reference Table

| Service | Type | CT/VM | IP | Port | URL | Tag |
|---------|------|-------|-----|------|-----|-----|
| n8n | LXC | 120 | 192.168.144.61 | 5678 | https://n8n.janvv.nl | Automation |
| ~~Syncthing~~ | ~~LXC~~ | ~~121~~ | ~~192.168.144.63~~ | ~~8384~~ | ~~https://sync.janvv.nl~~ | ~~Removed~~ |
| Planka | LXC | 122 | 192.168.144.60 | 1337 | https://tasks.janvv.nl | App |
| Grav | LXC | 123 | 192.168.144.72 | 80 | https://opa.janvv.nl | Web |
| AI Services | LXC | 124 | 192.168.144.62 | 11434/8080 | - | AI |
| Jellyfin | LXC | 125 | 192.168.144.100 | 8096 | https://kijkdoos.janvv.nl | Media |
| Cloudflared | LXC | 126 | 192.168.144.32 | - | - | Network |
| Lanproxy | LXC | 127 | 192.168.144.31 | 80/443 | - | Network |
| WordPress DB | LXC | 128 | 192.168.144.41 | 3306 | - | Database |
| WordPress jokegoudriaan | LXC | 129 | 192.168.144.70 | 80 | https://jokegoudriaan.nl | Web |
| WordPress kledingruil | LXC | 130 | 192.168.144.71 | 80 | https://kledingruil.jokegoudriaan.nl | Web |
| WordPress pgh | LXC | 131 | 192.168.144.73 | 80 | https://pgh.janvv.nl | Web |
| Immich | VM | 108 | 192.168.144.110 | 2283 | https://photos.janvv.nl | App |
| Home Assistant | VM | 114 | 192.168.144.120 | 8123 | https://assistant.janvv.nl | App |

## By Category

### Automation
- [[services/n8n|n8n]] - Workflow automation

### Applications
- [[services/planka|Planka]] - Project management
- [[services/immich|Immich]] - Photo management
- [[services/home-assistant|Home Assistant]] - Home automation

### AI & ML
- [[services/ai|AI Services]] - Ollama + Open WebUI

### Media
- [[services/jellyfin|Jellyfin]] - Media streaming

### Web & CMS
- [[services/grav|Grav]] - Flat-file CMS
- [[services/wordpress-jokegoudriaan|WordPress - jokegoudriaan.nl]]
- [[services/wordpress-kledingruil|WordPress - kledingruil]]
- [[services/wordpress-pgh|WordPress - pgh.janvv.nl]]

### Infrastructure
- [[services/wordpress-db|MariaDB]] - Shared database
- [[services/lanproxy|Lanproxy]] - Internal HTTPS
- [[services/cloudflared|Cloudflared]] - External access

## By Container ID

| CT | Service | Docs |
|----|---------|------|
| 108 | Immich (VM) | [[services/immich]] |
| 114 | Home Assistant (VM) | [[services/home-assistant]] |
| 120 | n8n | [[services/n8n]] |
| 122 | Planka | [[services/planka]] |
| 123 | Grav | [[services/grav]] |
| 124 | AI Services | [[services/ai]] |
| 125 | Jellyfin | [[services/jellyfin]] |
| 126 | Cloudflared | [[services/cloudflared]] |
| 127 | Lanproxy | [[services/lanproxy]] |
| 128 | WordPress DB | [[services/wordpress-db]] |
| 129 | WordPress jokegoudriaan | [[services/wordpress-jokegoudriaan]] |
| 130 | WordPress kledingruil | [[services/wordpress-kledingruil]] |
| 131 | WordPress pgh | [[services/wordpress-pgh]] |

## By IP Address

| IP | Service | Docs |
|-----|---------|------|
| 192.168.144.31 | Lanproxy | [[services/lanproxy]] |
| 192.168.144.32 | Cloudflared | [[services/cloudflared]] |
| 192.168.144.41 | WordPress DB | [[services/wordpress-db]] |
| 192.168.144.60 | Planka | [[services/planka]] |
| 192.168.144.61 | n8n | [[services/n8n]] |
| 192.168.144.62 | AI Services | [[services/ai]] |
| 192.168.144.70 | WordPress jokegoudriaan | [[services/wordpress-jokegoudriaan]] |
| 192.168.144.71 | WordPress kledingruil | [[services/wordpress-kledingruil]] |
| 192.168.144.72 | Grav | [[services/grav]] |
| 192.168.144.73 | WordPress pgh | [[services/wordpress-pgh]] |
| 192.168.144.100 | Jellyfin | [[services/jellyfin]] |
| 192.168.144.110 | Immich (VM) | [[services/immich]] |
| 192.168.144.120 | Home Assistant (VM) | [[services/home-assistant]] |

## Related

- [[README|Homelab Documentation Index]]
- [[infrastructure/network|Network Topology]]
