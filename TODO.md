# BitcoinDistrictRelay — Todo List

## 🏗️ Project Overview
BitcoinDistrictRelay is a fork of [nostrarabiarelay](https://github.com/tkhumush/nostrarabiarelay) — a Nostr relay (strfry) + Blossom media server deployment. It will be hosted on a separate server with its own domain, secrets, and community.

### Architecture
- **Strfry relay**: `relay.bitcoindistrict.org` (WebSocket + HTTPS)
- **Blossom server**: `cherry.bitcoindistrict.org` (media uploads, up to 100MB)
- **Main site**: `bitcoindistrict.org` (static landing page)
- **NIP-05**: `bitcoindistrict.org/.well-known/nostr.json` (Nostr identifier verification)
- **Reverse proxy**: Caddy (with automatic HTTPS)
- **Container orchestration**: Docker Compose

### ⚠️ MIGRATION — NOT FRESH DEPLOYMENT
This is a **migration** from a friend's existing server. The relay and blossom server are already running elsewhere with real data. Key implications:
- **Data migration required**: Copy ALL existing Nostr events from old strfry database to new server
- **Media migration required**: Copy ALL Blossom files from old server to new one
- **Zero-downtime cutover**: The friend will point DNS to our server ONLY AFTER we're set up and data is migrated
- **Sequence**: Set up new server → migrate data → test → friend switches DNS → verify

---

## ✅ Done
- [x] Fork nostrarabia codebase into BitcoinDistrictRelay
- [x] Global rebrand: nostrarabia → bitcoindistrict across all config files
- [x] Domain mapping: relay.bitcoindistrict.org, cherry.bitcoindistrict.org, bitcoindistrict.org
- [x] Docker image: ghcr.io/tkhumush/bitcoindistrictrelay
- [x] Fresh git init with main branch, remote set to GitHub
- [x] **Replace noteguard.toml whitelist** — 17 community pubkeys from bitcoindistrict.org
- [x] **Generate fresh secrets** — Created .env with random BLOSSOM_ADMIN_PASSWORD; created .env.example template
- [x] **Update SSL certificate paths** — Already correct (bitcoindistrict.org) in nginx and Caddy configs
- [x] **Customize strfry.conf** — Name, description, pubkey, contact email all set
- [x] **Customize blossom.yml** — publicDomain set; retention rules synced with whitelist (17 pubkeys, YAML anchor)
- [x] **Review Caddyfile** — All domains, reverse proxy rules, timeouts, security headers correct
- [x] **Update docker-compose.yml** — Container names, network name rebranded; blossom env references .env
- [x] **Add .env.example** — Template documenting BLOSSOM_ADMIN_PASSWORD
- [x] **Add .gitignore** — Excludes .env, data dirs, editor files
- [x] **Add HEALTH_CHECK.md** — Verification procedures for all services
- [x] **Fix index.html** — Replaced nostrarabia links with bitcoindistrict domains
- [x] **NIP-05 support** — Added `/.well-known/nostr.json` with CORS headers in Caddyfile; 17 community pubkeys mapped
- [x] **Community pubkeys synced** — noteguard.toml, blossom.yml, and nostr.json all use the same 17-member list

## 📋 To Do

### 🔴 High Priority (Before Deployment)
- [ ] **Update .github/workflows/deploy.yml** — Verify deployment target server, SSH keys, and paths. Current config references `/home/bitcoindistrict/nostr-relay` and GitHub secrets — confirm these match actual deployment server.
- [ ] **Update README.md** — Rewrite from scratch for BitcoinDistrictRelay (current README is well-done but needs final review for accuracy after all changes)
- [ ] **Provision deployment server** — Set up the new VPS/server where BitcoinDistrictRelay will run
- [ ] **Data migration: strfry database** — Document and execute migration of all existing Nostr events from the old server's strfry LMDB database to the new server
- [ ] **Data migration: Blossom media** — Document and execute migration of all existing media files (blobs + SQLite DB) from the old Blossom server to the new one
- [ ] **Migration checklist: DNS cutover** — Coordinate with the friend who currently hosts the relay. They will point DNS to our server only after we confirm data is migrated and services are tested. Document the exact sequence:
  1. Stop writes on old server (or coordinate a freeze window)
  2. Final rsync of data
  3. Friend changes DNS A records to point to new server IP
  4. Verify propagation
- [ ] **Post-migration verification** — Test relay connection, blossom uploads, NIP-05 resolution, WebSocket connectivity after DNS cutover

### 🟡 Medium Priority (Polish & Hardening)
- [ ] **Customize nginx configs** — Review `nginx/conf.d/*.conf` for completeness; they're currently correct but nginx is superseded by Caddy in the deployment
- [ ] **Expand noteguard/blossom/nostr.json** — Add new community members as they join (all three files must stay in sync)
- [ ] **Customize landing page** — `nginx/html/index.html` needs a proper Bitcoin District branded page (currently inherited from nostrarabia)

### 🟢 Nice to Have (Post-Launch)
- [ ] **Landing page** — Create a proper `index.html` for bitcoindistrict.org with Bitcoin District branding
- [ ] **NIP-11 relay info** — Verify NIP-11 document is complete and accurate in strfry.conf
- [ ] **Monitoring/alerting** — Set up health checks, uptime monitoring, and optional notifications
- [ ] **Backup strategy** — Document how to back up and restore the strfry database and blossom media storage
- [ ] **Contributing guide** — Add CONTRIBUTING.md if the project will be open to community contributions
- [ ] **Remove nginx configs** — Since Caddy is the primary proxy, consider removing `nginx/` directory to avoid confusion

---

## ⚠️ Important Notes
- **Do NOT push to GitHub** until Sir gives explicit approval
- **Do NOT reuse any secrets** from nostrarabiarelay — everything generated fresh
- The relay is whitelist-based (noteguard.toml) — manage pubkeys carefully
- **Three files must stay in sync** when adding/removing community members:
  1. `noteguard.toml` — relay write access
  2. `blossom.yml` — media upload/storage rules
  3. `nginx/html/.well-known/nostr.json` — NIP-05 identifiers
- Blossom server allows 100MB uploads — ensure adequate disk space on deployment server
- Caddy handles SSL automatically — no certbot or manual certificate management needed
- The `.env` file contains the real blossom admin password and must NEVER be committed