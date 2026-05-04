# BitcoinDistrictRelay

A whitelisted Nostr relay powered by [strfry](https://github.com/hoytech/strfry) with [noteguard](https://github.com/damus-io/noteguard) for access control, plus [blossom-server](https://github.com/hzrd149/blossom-server) for media storage.

## Architecture

| Service | Domain | Port | Purpose |
|---------|--------|------|---------|
| strfry relay | `relay.bitcoindistrict.org` | 7777 | Nostr relay (WebSocket) |
| blossom-server | `cherry.bitcoindistrict.org` | 3000 | Media/blob uploads (50MB max) |
| caddy | — | 80/443 | Reverse proxy + auto HTTPS |

> **Note:** The apex domain `bitcoindistrict.org` and NIP-05 endpoint are served externally — not by this relay's Caddy.

## ⚠️ Migration Project

This is a **migration** from a friend's existing server, not a fresh deployment. See [MIGRATION.md](MIGRATION.md) for the step-by-step data migration guide.

## Quick Start

### Prerequisites

- Docker with Compose plugin (v2)
- Git

### Setup

1. Clone and configure:
```bash
git clone https://github.com/tkhumush/bitcoindistrictrelay.git
cd bitcoindistrictrelay

# Create secrets file
cp .env.example .env
# Edit .env — set BLOSSOM_ADMIN_PASSWORD
nano .env
```

2. Customize the whitelist in `noteguard.toml` — add your community pubkeys in hex format.

3. Start services:
```bash
docker compose up -d
```

4. Verify:
```bash
# Relay NIP-11
curl http://localhost:7777 -H "Accept: application/nostr+json"

# Blossom health
curl http://localhost:3000/
```

Local access: `ws://localhost:7777` (relay) · `http://localhost:3000` (blossom)

## Configuration

### Adding Whitelisted Users

Edit `noteguard.toml` — pubkeys must be in hex (not npub). Three files must stay in sync:

1. **`noteguard.toml`** — relay write access
2. **`blossom.yml`** — media upload/storage rules (YAML anchor `&community`)
3. **`nginx/html/.well-known/nostr.json`** — NIP-05 identifiers (served externally)

After changes: `docker compose restart nostr-relay`

### Converting npub to hex

```bash
# Using nak CLI
go install github.com/fiatjaf/nak@latest
nak decode npub1...

# Online: https://nostr.band
```

### Relay Settings (`strfry.conf`)

- `relay.info.name` / `.description` — NIP-11 relay info
- `relay.info.pubkey` — admin contact pubkey
- `events.maxEventSize` — max event size (64KB default)
- `dbParams.mapsize` — LMDB size (10GB default)
- `relay.nofiles` — set to `0` for container compatibility

### Blossom Settings (`blossom.yml`)

- `publicDomain` — domain for blob URLs (`cherry.bitcoindistrict.org`)
- `storage.rules` — retention by pubkey + file type (100yr = effectively permanent)
- `upload.requireAuth` — require Nostr auth to upload
- `media.image` / `media.video` — optimization settings

## Deployment

### CI/CD (GitHub Actions)

Pushing to `main` triggers:
1. Docker image build → `ghcr.io/tkhumush/bitcoindistrictrelay`
2. SSH deployment to server (requires GitHub secrets: `DEPLOY_HOST`, `DEPLOY_USER`, `DEPLOY_SSH_KEY`)

### Manual Deployment

```bash
# On server
git clone https://github.com/tkhumush/bitcoindistrictrelay.git
cd bitcoindistrictrelay
cp .env.example .env  # Set BLOSSOM_ADMIN_PASSWORD
docker compose pull nostr-relay
docker compose up -d
```

Caddy automatically obtains SSL certificates on first request (~30-60 seconds).

### DNS Configuration

```
relay.bitcoindistrict.org    →  NEW_SERVER_IP
cherry.bitcoindistrict.org   →  NEW_SERVER_IP
```

Ports 80 and 443 must be open for Caddy's automatic HTTPS.

## Monitoring

```bash
# Logs
docker compose logs -f nostr-relay    # relay
docker compose logs -f blossom-server  # media
docker compose logs -f caddy           # proxy

# Status
docker compose ps

# NIP-11 (production)
curl -s https://relay.bitcoindistrict.org -H "Accept: application/nostr+json" | jq .

# Caddy config validation
docker compose exec caddy caddy validate --config /etc/caddy/Caddyfile
```

See [HEALTH_CHECK.md](HEALTH_CHECK.md) for full verification procedures.

## Troubleshooting

| Symptom | Fix |
|---------|-----|
| 502 Bad Gateway | `docker compose restart nostr-relay blossom-server` |
| Events rejected | Pubkey not in `noteguard.toml` whitelist |
| SSL cert failure | DNS A records must point to server IP; ports 80/443 open |
| Upload fails | File exceeds 50MB limit |
| nofiles error | `nofiles=0` in strfry.conf (already set) |

More in [HEALTH_CHECK.md](HEALTH_CHECK.md).

## Data Migration

See [MIGRATION.md](MIGRATION.md) for the complete migration guide including:
- Strfry LMDB database migration
- Blossom media file migration
- DNS cutover coordination
- Post-migration verification

## Backup

```bash
# Strfry database
docker compose exec nostr-relay tar czf - /app/strfry-db > backup-strfry-$(date +%Y%m%d).tar.gz

# Blossom data (blobs + SQLite)
docker compose exec blossom-server tar czf - /app/data > backup-blossom-$(date +%Y%m%d).tar.gz

# Caddy certificates
docker run --rm -v bitcoindistrictrelay_caddy-data:/data -v $(pwd):/backup alpine \
  tar czf /backup/backup-caddy-$(date +%Y%m%d).tar.gz -C /data .
```

## Resources

- [Strfry](https://github.com/hoytech/strfry)
- [Noteguard](https://github.com/damus-io/noteguard)
- [Blossom Server](https://github.com/hzrd149/blossom-server)
- [Blossom Protocol](https://github.com/hzrd149/blossom)
- [Nostr Protocol](https://github.com/nostr-protocol/nostr)
- [NIPs](https://github.com/nostr-protocol/nips)

## License

Configuration is provided as-is. See individual component licenses (strfry, noteguard, blossom-server).# Last updated: 2026-05-04T12:16:33Z
