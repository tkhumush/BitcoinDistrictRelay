# Migration Guide — BitcoinDistrictRelay

This guide covers migrating the BitcoinDistrictRelay from the current (friend's) server to the new server. The relay and blossom server are already running with real data — this must be done carefully to avoid data loss.

## Prerequisites

- New server provisioned with Docker + Compose plugin
- SSH access to both old and new servers
- DNS credentials or coordination with the friend who controls DNS
- The `.env` file with `BLOSSOM_ADMIN_PASSWORD` on the new server

## Phase 1: Set Up New Server (No Traffic)

```bash
# On the NEW server
git clone https://github.com/tkhumush/bitcoindistrictrelay.git
cd bitcoindistrictrelay

# Create .env from .env.example
cp .env.example .env
# Edit .env with the real BLOSSOM_ADMIN_PASSWORD

# Start services (they'll be empty but running)
docker compose up -d

# Verify services are healthy
docker compose ps
```

### Verify New Server (Before Data)

```bash
# Relay NIP-11 should respond
curl -s http://localhost:7777 -H "Accept: application/nostr+json"

# Blossom healthcheck
docker compose exec blossom-server wget --spider -q http://localhost:3000/

# Caddy should be ready for SSL (will get certs on first real request)
docker compose logs caddy --tail 20
```

## Phase 2: Migrate Strfry Database

Strfry uses LMDB. The safest migration method is to copy the database files while strfry is NOT writing to them.

### Option A: Stop-then-Copy (Recommended — Zero Data Loss)

```bash
# On the OLD server — stop the relay
docker compose stop nostr-relay
# OR: docker stop <relay-container-name>

# Copy the LMDB database to the new server
# The database is typically at /app/strfry-db inside the container,
# mounted as a Docker volume
rsync -avz --progress \
  /path/to/old/strfry-db/ \
  newserver:/path/to/bitcoindistrictrelay/strfry-db/

# On the OLD server — restart the relay (it will resume receiving events)
docker compose start nostr-relay
```

### Option B: Live Copy with Catch-up (Minimal Downtime)

```bash
# 1. First rsync while relay is running (warm copy)
rsync -avz --progress \
  /path/to/old/strfry-db/ \
  newserver:/path/to/bitcoindistrictrelay/strfry-db/

# 2. Brief stop for final sync
# On OLD server:
docker compose stop nostr-relay

# 3. Second rsync (only changed files — fast)
rsync -avz --progress \
  /path/to/old/strfry-db/ \
  newserver:/path/to/bitcoindistrictrelay/strfry-db/

# 4. Immediately restart old relay to minimize write gap
docker compose start nostr-relay
```

### Strfry Volume Location

Find the actual volume path on the old server:
```bash
# List volumes
docker volume ls | grep strfry

# Inspect volume mount point
docker volume inspect <volume-name> --format '{{.Mountpoint}}'
```

On the new server, the strfry data volume is named `bitcoindistrictrelay_strfry-data` (or per docker-compose). You can either:
1. Copy directly into the volume mount point, OR
2. Mount a host directory instead of a named volume (edit docker-compose.yml)

**To use a host directory** (easier for migration):
```yaml
# In docker-compose.yml, change:
- strfry-data:/app/strfry-db
# To:
- ./strfry-db:/app/strfry-db
```

Then create the directory and copy data into `./strfry-db/`.

## Phase 3: Migrate Blossom Media

Blossom stores files in two locations:
1. **SQLite database**: `data/sqlite.db` (blob metadata)
2. **Blob files**: `data/blobs/` (actual uploaded files)

### Migration Steps

```bash
# On the OLD server — find the blossom data volume
docker volume inspect <blossom-volume-name> --format '{{.Mountpoint}}'

# Copy the entire data directory
rsync -avz --progress \
  /path/to/old/blossom-data/ \
  newserver:/path/to/bitcoindistrictrelay/blossom-data/
```

**To use a host directory** (easier for migration):
```yaml
# In docker-compose.yml, change:
- blossom-data:/app/data
# To:
- ./blossom-data:/app/data
```

Then: `mkdir -p blossom-data && rsync` into it.

> **Important**: The blossom container must be STOPPED during rsync if the SQLite DB is being written to. Otherwise you risk a corrupted database.

## Phase 4: Pre-Cutover Verification

Before switching DNS, verify the new server works with the migrated data:

```bash
# On the NEW server
docker compose restart nostr-relay blossom-server

# Check relay has events
echo '["REQ","verify",{"limit":5}]' | websocat ws://localhost:7777

# Check blossom has blobs
curl -s http://localhost:3000/ | head -20

# Full smoke test (see HEALTH_CHECK.md)
```

Optionally, test via `/etc/hosts` override:
```bash
# On your LOCAL machine (not the server)
echo "NEW_SERVER_IP relay.bitcoindistrict.org" | sudo tee -a /etc/hosts
echo "NEW_SERVER_IP cherry.bitcoindistrict.org" | sudo tee -a /etc/hosts

# Test with a Nostr client pointing to wss://relay.bitcoindistrict.org
# Test blossom upload at https://cherry.bitcoindistrict.org

# Clean up when done
sudo sed -i '/relay.bitcoindistrict.org/d' /etc/hosts
sudo sed -i '/cherry.bitcoindistrict.org/d' /etc/hosts
```

## Phase 5: DNS Cutover

**Coordinate with the friend who currently controls DNS.**

### Cutover Sequence

1. **Notify**: Tell the friend you're ready for the switch
2. **Final sync**: Run a final rsync to catch any events that happened since the last sync
   ```bash
   # Stop old relay briefly
   # rsync strfry-db
   # rsync blossom-data
   # Restart old relay
   ```
3. **DNS change**: Friend updates A records:
   - `relay.bitcoindistrict.org` → NEW_SERVER_IP
   - `cherry.bitcoindistrict.org` → NEW_SERVER_IP
   - (Apex `bitcoindistrict.org` stays wherever it is)
4. **Wait for propagation**: DNS TTL is typically 5min–1hr
5. **Verify**: Test from an external network

### Post-Cutover Verification

```bash
# From an external machine (not the server itself)
# Relay NIP-11
curl -s https://relay.bitcoindistrict.org -H "Accept: application/nostr+json" | jq .

# WebSocket test
echo '["REQ","post-cut",{"limit":1}]' | websocat wss://relay.bitcoindistrict.org

# Blossom dashboard
curl -sf https://cherry.bitcoindistrict.org/ && echo "OK"

# Publish a test event (must be a whitelisted pubkey)
# Verify it appears in the relay
```

## Phase 6: Monitor

For the first 24-48 hours after cutover:

- Watch relay logs: `docker compose logs -f nostr-relay`
- Watch blossom logs: `docker compose logs -f blossom-server`
- Watch Caddy logs: `docker compose logs -f caddy`
- Monitor disk usage: `df -h`
- Check for rejected events in noteguard logs

## Rollback Plan

If something goes wrong after DNS cutover:

1. Friend changes DNS A records back to the OLD server IP
2. Users automatically reconnect to the old server as DNS propagates
3. Any events published to the NEW server during the outage can be exported and imported to the old server using strfry's `export`/`import` commands

## Appendix: strfry DB Export/Import

If you need to export/import individual events (e.g., for catch-up sync):

```bash
# Export all events from old server
docker compose exec nostr-relay ./strfry export > events.jsonl

# Import events into new server
cat events.jsonl | docker compose exec -T nostr-relay ./strfry import

# Or export a specific range
docker compose exec nostr-relay ./strfry export --since 1700000000 > recent.jsonl
```