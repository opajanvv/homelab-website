---
type: how-to
tags: [troubleshooting, debugging, logs]
last-tested: 2026-02-06
---

# How to Troubleshoot Service Issues

Common problems and solutions for homelab services.

## When to use

Service not working, container won't start, or unexpected behavior.

## Diagnostic Steps

### 1. Check container status

```bash
# Is container running?
pct status <CT_ID>

# If not running, start it
pct start <CT_ID>
```

### 2. Check Docker status

```bash
# Is Docker running?
pct exec <CT_ID> -- systemctl status docker

# If not, start it
pct exec <CT_ID> -- systemctl start docker
```

### 3. Check Docker containers

```bash
# List containers
pct exec <CT_ID> -- docker ps -a

# Are containers running?
# If not, check why
```

### 4. View logs

```bash
# Docker Compose logs
pct exec <CT_ID> -- bash -c 'cd /opt/homelab-docker/<service> && docker compose logs'

# Follow logs
pct exec <CT_ID> -- bash -c 'cd /opt/homelab-docker/<service> && docker compose logs -f'

# Specific container logs
pct exec <CT_ID> -- docker logs <container-name>
```

### 5. Check data directory

```bash
# Does data directory exist?
ls -la /lxcdata/<service>/

# Are permissions correct?
ls -la /lxcdata/<service>/
```

## Common Issues

### Container won't start

**Symptoms:** `pct start` fails or container stops immediately

**Solutions:**

1. **Check container config:**
   ```bash
   cat /etc/pve/lxc/<CT_ID>.conf
   ```

2. **Check for AppArmor issues:**
   ```bash
   # Verify AppArmor workaround is present
   grep apparmor /etc/pve/lxc/<CT_ID>.conf
   ```

3. **Check container logs:**
   ```bash
   journalctl -xe -u pve-container@<CT_ID>
   ```

### Service containers not running

**Symptoms:** `docker ps` shows no or stopped containers

**Solutions:**

1. **Check Docker Compose config:**
   ```bash
   pct exec <CT_ID> -- bash -c 'cd /opt/homelab-docker/<service> && docker compose config'
   ```

2. **Check for missing .env file:**
   ```bash
   pct exec <CT_ID> -- ls -la /opt/homelab-docker/<service>/.env
   ```

3. **Try starting manually:**
   ```bash
   pct exec <CT_ID> -- bash -c 'cd /opt/homelab-docker/<service> && docker compose up -d'
   ```

### Permission issues

**Symptoms:** "Permission denied" errors, service can't write to `/data`

**Cause:** Unprivileged LXC containers map UIDs with +100000 offset

**Fix:**
```bash
# Find correct ownership (example for www-data, UID 33)
chown -R 100033:100033 /lxcdata/<service>/<dir>

# Common mappings:
# UID 33 (www-data) → Host UID 100033
# UID 70 (postgres) → Host UID 100070
# UID 999 (mysql) → Host UID 100999
```

### Network issues

**Symptoms:** Can't access service from other machines

**Diagnosis:**
```bash
# Test from Proxmox host
curl http://<service-ip>:<port>

# Test from inside container
pct exec <CT_ID> -- curl localhost:<port>

# Check DNS
dig @192.168.144.20 <domain>
```

**Solutions:**

1. **Check firewall rules on container**
2. **Check routing in Cloudflare/Lanproxy**
3. **Verify service is listening**
   ```bash
   pct exec <CT_ID> -- netstat -tlnp
   ```

### Out of disk space

**Symptoms:** "No space left on device" errors

**Diagnosis:**
```bash
# Check container disk usage
pct exec <CT_ID> -- df -h

# Check data directory size
du -sh /lxcdata/<service>
```

**Solutions:**

1. **Clean Docker images:**
   ```bash
   pct exec <CT_ID> -- docker system prune -a
   ```

2. **Clean logs if oversized**
3. **Expand disk if needed**

### Database connection errors

**Symptoms:** Service can't connect to database

**Diagnosis:**
```bash
# From service container, can you reach DB?
pct exec <SERVICE_CT> -- curl <db-ip>:<db-port>

# Is DB container running?
pct exec <DB_CT> -- docker ps
```

**Solutions:**

1. **Check database credentials in .env**
2. **Verify database is running**
3. **Check database logs**

## Getting Help

When asking for help, provide:

1. **Service name and CT ID**
2. **Symptoms** - what's not working
3. **Steps taken** - what you've already tried
4. **Relevant logs** - error messages
5. **Container status** - `pct status <CT_ID>` and `docker ps -a`

## Related

- [[./access-container|Access Containers]]
- [[./backup-restore|Backup & Restore]]
- [[../infrastructure/proxmox|Proxmox Configuration]]
