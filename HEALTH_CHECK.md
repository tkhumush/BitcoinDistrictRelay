# Health Check Guide — BitcoinDistrictRelay

## Quick Status Check

```bash
# Check all containers are running
docker compose ps

# Expected: 3 services Up (bitcoindistrict-relay, bitcoindistrict-blossom, bitcoindistrict-caddy)
```

## Relay Health (strfry)

```bash
# WebSocket connectivity test (production)
echo '["REQ","health",{"limit":1}]' | websocat wss://relay.bitcoindistrict.org

# NIP-11 relay info
curl -s https://relay.bitcoindistrict.org -H "Accept: application/nostr+json" | jq .

# Should return JSON with:
#   name: "Bitcoin District Relay"
#   description: "A whitelisted nostr relay for the Bitcoin District community"
#   pubkey: "9cb3545c36940d9a2ef86d50d5c7a8fab90310cc898c4344bcfc4c822ff47bca"

# Container health check (local)
docker compose exec nostr-relay curl -sf http://localhost:7777/ && echo "OK" || echo "FAIL"
```

## Blossom Server Health (cherry.bitcoindistrict.org)

```bash
# HTTPS connectivity
curl -sf https://cherry.bitcoindistrict.org/ && echo "OK" || echo "FAIL"

# Admin dashboard (requires password from .env)
curl -sf https://cherry.bitcoindistrict.org/ -u admin:$(grep BLOSSOM_ADMIN_PASSWORD .env | cut -d= -f2)

# Container health check (local)
docker compose exec blossom-server wget --spider -q http://localhost:3000/ && echo "OK" || echo "FAIL"
```

## Caddy Proxy Health

```bash
# Check Caddy is proxying correctly
docker compose logs caddy --tail 20

# Validate Caddyfile syntax
docker compose exec caddy caddy validate --config /etc/caddy/Caddyfile

# Reload Caddy configuration (without restart)
docker compose exec caddy caddy reload --config /etc/caddy/Caddyfile
```

## Full Smoke Test (Local)

```bash
# 1. All containers up
docker compose ps | grep -c "Up" | grep -q 3 && echo "✅ All containers running" || echo "❌ Container issue"

# 2. Relay responds
curl -sf http://localhost:7777 -H "Accept: application/nostr+json" > /dev/null && echo "✅ Relay healthy" || echo "❌ Relay issue"

# 3. Blossom responds
curl -sf http://localhost:3000/ > /dev/null && echo "✅ Blossom healthy" || echo "❌ Blossom issue"
```

## Full Smoke Test (Production — After DNS Cutover)

```bash
# 1. Relay NIP-11
curl -sf https://relay.bitcoindistrict.org -H "Accept: application/nostr+json" > /dev/null && echo "✅ Relay healthy" || echo "❌ Relay issue"

# 2. Blossom responds
curl -sf https://cherry.bitcoindistrict.org/ > /dev/null && echo "✅ Blossom healthy" || echo "❌ Blossom issue"

# 3. WebSocket works
echo '["REQ","smoke",{"limit":1}]' | websocat wss://relay.bitcoindistrict.org
```

## Troubleshooting

| Symptom | Likely Cause | Fix |
|---------|-------------|-----|
| 502 Bad Gateway | Backend container not ready | `docker compose restart nostr-relay blossom-server` |
| SSL cert failure | DNS not pointed to server | Verify A records point to server IP |
| Events rejected | Pubkey not in whitelist | Add to `noteguard.toml` and restart |
| Upload fails (>50MB) | Exceeds Caddy/blossom limit | Files must be under 50MB |
| Blossom auth error | Wrong admin password | Check `BLOSSOM_ADMIN_PASSWORD` in `.env` |
| Caddy won't start | Port 80/443 in use | Stop nginx: `systemctl stop nginx` |
| nofiles error | Container lacks permission | `nofiles=0` in strfry.conf (already set) |

## Note

The apex domain `bitcoindistrict.org` and NIP-05 endpoint are served externally — they are NOT handled by this relay's Caddy. Health checks here only cover `relay.*` and `cherry.*`.