# BitcoinDistrictRelay — Todo List

## 🏗️ Project Overview
BitcoinDistrictRelay is a fork of [nostrarabiarelay](https://github.com/tkhumush/nostrarabiarelay) — a Nostr relay (strfry) + Blossom media server deployment. It will be hosted on a separate server with its own domain, secrets, and community.

### Architecture
- **Strfry relay**: `relay.bitcoindistrict.org` (WebSocket + HTTPS)
- **Blossom server**: `cherry.bitcoindistrict.org` (media uploads, up to 100MB)
- **Main site**: `bitcoindistrict.org` (static landing page)
- **Reverse proxy**: Caddy (with automatic HTTPS)
- **Container orchestration**: Docker Compose

---

## ✅ Done
- [x] Fork nostrarabia codebase into BitcoinDistrictRelay
- [x] Global rebrand: nostrarabia → bitcoindistrict across all config files
- [x] Domain mapping: relay.bitcoindistrict.org, cherry.bitcoindistrict.org, bitcoindistrict.org
- [x] Docker image: ghcr.io/tkhumush/bitcoindistrictrelay
- [x] Fresh git init with main branch, remote set to GitHub
- [x] **Replace noteguard.toml whitelist** — Trimmed to Bitcoin District members (tkay, bitcoindistrict, AlexBarq); added header comments with instructions
- [x] **Generate fresh secrets** — Created .env with random BLOSSOM_ADMIN_PASSWORD; created .env.example template
- [x] **Update SSL certificate paths** — Already correct (bitcoindistrict.org) in nginx and Caddy configs
- [x] **Customize strfry.conf** — Name, description, pubkey, contact email all set
- [x] **Customize blossom.yml** — publicDomain set to cherry.bitcoindistrict.org; retention rules synced with whitelist
- [x] **Review Caddyfile** — All domains, reverse proxy rules, timeouts, security headers correct
- [x] **Update docker-compose.yml** — Container names, network name rebranded; blossom env references .env
- [x] **Add .env.example** — Template documenting BLOSSOM_ADMIN_PASSWORD
- [x] **Add .gitignore** — Excludes .env, data dirs, editor files
- [x] **Add HEALTH_CHECK.md** — Verification procedures for all services
- [x] **Fix index.html** — Replaced nostrarabia links with bitcoindistrict domains

## 📋 To Do

### 🔴 High Priority (Before Deployment)
- [ ] **Update .github/workflows/deploy.yml** — Verify deployment target server, SSH keys, and paths. Current config references `/home/bitcoindistrict/nostr-relay` and GitHub secrets — confirm these match actual deployment server.
- [ ] **Update README.md** — Rewrite from scratch for BitcoinDistrictRelay (current README is well-done but needs final review for accuracy after all changes)

### 🟡 Medium Priority (Polish & Hardening)
- [ ] **Customize nginx configs** — Review `nginx/conf.d/*.conf` for completeness; they're currently correct but nginx is superseded by Caddy in the deployment
- [ ] **Expand noteguard whitelist** — Add Bitcoin District community members as they're identified (currently only 3 pubkeys)
- [ ] **Expand blossom retention rules** — Sync pubkeys as whitelist grows
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
- Blossom server allows 100MB uploads — ensure adequate disk space on deployment server
- Caddy handles SSL automatically — no certbot or manual certificate management needed
- The `.env` file contains the real blossom admin password and must NEVER be committed