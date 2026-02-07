---
type: how-to
tags: [backup, restore, disaster-recovery]
last-tested: 2026-02-06
---

# How to Backup and Restore

Procedures for backing up and restoring homelab services.

## When to use

- Regular backup maintenance
- Before making major changes
- Recovering from data loss
- Migrating to new hardware

## Prerequisites

- [ ] Backup destination available (external drive, network storage, etc.)
- [ ] Know which services to backup
- [ ] Database credentials (for DB backups)

## Service Data Backups

### File-based backups

Most service data is in `/lxcdata/<service>/`:

```bash
# Backup specific service
rsync -av /lxcdata/<service>/ /backup/homelab/<service>/$(date +%F)/

# Backup all services
for service in /lxcdata/*; do
    name=$(basename "$service")
    rsync -av "$service/" "/backup/homelab/$name/$(date +%F)/"
done
```

### ZFS snapshots

If using ZFS for `/lxcdata`:

```bash
# Snapshot before changes
zfs snapshot zpool/lxcdata/<service>@before-$(date +%F)

# List snapshots
zfs list -t snapshot

# Restore from snapshot (destructive)
zfs rollback zpool/lxcdata/<service>@snapshot-name

# Clone snapshot (non-destructive)
zfs clone zpool/lxcdata/<service>@snapshot-name zpool/lxcdata/<service>-restore
```

## Database Backups

### MariaDB (WordPress)

**Backup:**
```bash
# All databases
pct exec 128 -- bash -c 'docker exec mariadb mysqldump -u root -p --all-databases' > mariadb-backup-$(date +%F).sql

# Single database
pct exec 128 -- bash -c 'docker exec mariadb mysqldump -u root -p wordpress' > wordpress-backup-$(date +%F).sql
```

**Restore:**
```bash
# All databases
cat mariadb-backup.sql | pct exec 128 -- bash -c 'docker exec -i mariadb mysql -u root -p'

# Single database
cat wordpress-backup.sql | pct exec 128 -- bash -c 'docker exec -i mariadb mysql -u root -p wordpress'
```

### PostgreSQL (Planka)

**Backup:**
```bash
pct exec 122 -- bash -c 'docker exec postgres pg_dumpall -U planka' > planka-backup-$(date +%F).sql
```

**Restore:**
```bash
cat planka-backup.sql | pct exec 122 -- bash -c 'docker exec -i postgres psql -U planka'
```

## Container Configuration Backups

**LXC configs:**
```bash
# Backup all LXC configs
for ct in /etc/pve/lxc/*.conf; do
    cp "$ct" /backup/homelab/lxc-configs/
done
```

**VM configs:**
```bash
# Backup all VM configs
for vm in /etc/pve/qemu-server/*.conf; do
    cp "$vm" /backup/homelab/vm-configs/
done
```

## Complete Service Restore

### 1. Restore data

```bash
# Stop container
pct stop <CT_ID>

# Restore data
rsync -av /backup/homelab/<service>/ /lxcdata/<service>/

# Start container
pct start <CT_ID>
```

### 2. Restore database (if applicable)

See database restore sections above.

### 3. Verify

```bash
# Check service status
pct exec <CT_ID> -- bash -c 'cd /opt/homelab-docker/<service> && docker compose ps'

# View logs
pct exec <CT_ID> -- bash -c 'cd /opt/homelab-docker/<service> && docker compose logs'
```

## Verification

- [ ] Backup files exist at destination
- [ ] Backup files are not corrupted (check file sizes)
- [ ] Test restore procedure periodically
- [ ] Document any restore issues

## Troubleshooting

**Restore fails permissions issue:**
```bash
# Fix ownership for unprivileged LXC
chown -R 100033:100033 /lxcdata/<service>  # for www-data (UID 33)
```

**Database connection errors after restore:**
- Verify database credentials in .env files
- Check database container is running
- Restart application container

## Related

- [[../infrastructure/storage|Storage Configuration]]
- [[./restore-from-scratch|Complete Disaster Recovery]]
