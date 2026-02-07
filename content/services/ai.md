---
service: AI Services
status: running
host: 192.168.144.62
ct: 124
port: 11434/8080
tags: [ai, llm, ollama]
updated: 2026-02-06
---

# AI Services (Ollama + Open WebUI)

Local AI services for running Large Language Models.

## Overview

Combines Ollama (LLM runner) with Open WebUI (ChatGPT-like interface) for local AI capabilities.

**Access:** http://192.168.144.62:8080 (Open WebUI)
**Container:** CT 124
**IP:** 192.168.144.62
**Ports:** 11434 (Ollama API), 8080 (Open WebUI)

## Source Code

**Docker Compose & Install Script:** [github.com/opajanvv/homelab-docker/tree/main/ai](https://github.com/opajanvv/homelab-docker/tree/main/ai)
**Local copy:** `~/dev/homelab-docker/ai/`

## Deployment

```bash
# Clone LXC template
pct clone 902 124 --hostname ai --full
pct set 124 --cores 4 --memory 8192
pct set 124 -net0 name=eth0,bridge=vmbr0,firewall=1,gw=192.168.144.1,ip=192.168.144.62/23
pct set 124 -mp0 /lxcdata/ai,mp=/data
pct set 124 -features nesting=1,keyctl=1
pct set 124 -onboot 1

# Add AppArmor workaround
cat >> /etc/pve/lxc/124.conf << 'EOF'
lxc.apparmor.profile: unconfined
lxc.mount.entry: /dev/null sys/module/apparmor/parameters/enabled none bind 0 0
EOF

# Deploy
pct start 124
pct exec 124 -- bash -c 'systemctl enable --now docker'
pct exec 124 -- bash -c 'git clone https://github.com/opajanvv/homelab-docker.git /opt/homelab-docker'
pct exec 124 -- bash -c 'cd /opt/homelab-docker/ai && chmod +x install.sh && ./install.sh'
```

## Configuration

**Stack:** Ollama + Open WebUI

**Data Locations:**
- `/data/ollama/` - Downloaded models (inside container)
- `/opt/homelab-docker/ai/` - Docker Compose config (inside container)
- `~/dev/homelab-docker/ai/` - Local working directory (laptop)

**Resources:**
- Needs more CPU and RAM than other services (4 cores, 8GB recommended)
- Models vary in size; large models need more resources

## Access

- **Open WebUI:** http://192.168.144.62:8080
- **Ollama API:** http://192.168.144.62:11434

**First Use:** Open WebUI, create account, then pull models via UI

## Models

Models are downloaded on-demand and stored in `/data/ollama/`.

**Common models:**
- `llama2` - Meta's LLaMA 2
- `mistral` - Mistral AI
- `codellama` - Code-specialized

**Pull via WebUI:** Settings → Models → Pull Model
**Pull via CLI (in container):**
```bash
pct exec 124 -- bash -c 'docker exec ollama ollama pull <model-name>'
```

## Backup

**What to backup:**
- `/lxcdata/ai/ollama/` - Downloaded models (can be large)

**Backup command:**
```bash
rsync -av /lxcdata/ai/ollama/ /backup/homelab/ai-models/
```

**Note:** Models can be re-downloaded, but backing up saves time/bandwidth.

## Maintenance

**Update:**
```bash
pct exec 124 -- bash -c 'cd /opt/homelab-docker/ai && docker compose pull && docker compose up -d'
```

**View logs:**
```bash
pct exec 124 -- bash -c 'cd /opt/homelab-docker/ai && docker compose logs -f'
```

**List models:**
```bash
pct exec 124 -- bash -c 'docker exec ollama ollama list'
```

**Remove unused models:**
```bash
pct exec 124 -- bash -c 'docker exec ollama ollama rm <model-name>'
```

## Common Tasks

**Chat:** Via Open WebUI interface
**Pull new model:** Settings → Models → Pull
**Switch model:** Model selector in chat interface

## Related

- [[infrastructure/network|Network routing]]
- [[how-to/update-service|Update a Service]]
