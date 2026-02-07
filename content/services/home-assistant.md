---
service: Home Assistant
status: running
host: 192.168.144.120
vm: 114
port: 8123
tags: [app, home-automation, smart-home]
updated: 2026-02-06
---

# Home Assistant

Home automation platform.

## Overview

Home Assistant integrates with smart home devices and automations. Open source alternative to commercial smart home platforms.

**Access:** https://assistant.janvv.nl
**VM:** 114
**IP:** 192.168.144.120
**Port:** 8123
**Resources:** 2 cores, 8GB RAM, 32GB disk

## Deployment

Home Assistant runs on a dedicated VM.

```bash
# Create VM
qm create 114 --name home-assistant --cores 2 --memory 8192 --net0 virtio,bridge=vmbr0
# ... additional VM configuration

# Install Home Assistant OS
# Follow Home Assistant installation guide
```

## Configuration

Home Assistant typically uses Home Assistant OS on the VM, which is an all-in-one OS + Home Assistant.

**Configuration:**
- `configuration.yaml` - Main configuration
- Automations via UI
- Integrations added via UI

## Access

- **Web UI:** https://assistant.janvv.nl
- **Direct:** http://192.168.144.120:8123
- **Mobile Apps:** Available for iOS and Android

## Integrations

Common integrations include:
- Smart lights (Hue, Zigbee)
- Sensors (temperature, motion)
- Media devices
- Weather services
- Calendar services

Add via: Settings → Devices & Services → Add Integration

## Automation

**Automations:** Create via UI (Settings → Automations & Scenes)
**Scripts:** Reusable action sequences
**Scenes:** Pre-defined device states

## Backup

**Home Assistant backups:**
- Automatic backups scheduled in UI
- Manual backups: Settings → System → Backups → Create Backup
- Backups stored within VM, should copy off-VM for safety

**Full backup:** Snapshot the VM itself

## Maintenance

**Update:** Via UI (Settings → System → Updates)
**Restart:** Via UI or restart VM

## Common Tasks

**Add device:** Settings → Devices & Services → Add Integration
**Create automation:** Settings → Automations & Scenes → Create Automation
**Check logs:** Settings → System → Logs

## Related

- [[infrastructure/network|Network routing]]
