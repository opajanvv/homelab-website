---
service: Syncthing
status: removed
ct: 121 (deleted)
port: 8384
tags: [app, file-sync, backup]
updated: 2026-02-07
---

# Syncthing

**⚠️ REMOVED** - This service has been decommissioned. CT 121 no longer exists.

## Historical Overview

Syncthing provided peer-to-peer file synchronization between devices.

**Former Access:** https://sync.janvv.nl
**Former Container:** CT 121 (deleted)
**Former IP:** 192.168.144.63

## Cleanup Required

The following data directory still exists and should be cleaned up:

```bash
# Remove orphaned data directory
rm -rf /lxcdata/syncthing/
```

## Related

- [[how-to/deploy-new-service|Deploy New Service]]
