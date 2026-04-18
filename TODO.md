# BitcoinDistrictRelay — Todo List

## 🏗️ Project Overview
BitcoinDistrictRelay is a fork of [nostrarabiarelay](https://github.com/tkhumush/nostrarabiarelay) — a Nostr relay (strfry) + Blossom media server deployment. It will be hosted on a separate server with its own domain, secrets, and community.

### Architecture
- **Strfry relay**: `relay.bitcoindistrict.org` (WebSocket + HTTPS)
- **Blossom server**: `cherry.bitcoindistrict.org` (media uploads, up to 100MB)
- **Main site**: `bitcoindistrict.org` (static landing page)
- **Reverse proxy**: Caddy (with automatic HTTPS) or Nginx (manual SSL)
- **Container orchestration**: Docker Compose

---

## ✅ Done
- [x] Fork nostrarabiarelay codebase into BitcoinDistrictRelay
- [x] Global rebrand: nostrarabia → bitcoindistrict across all config files
- [x] Domain mapping: relay.bitcoindistrict.org, cherry.bitcoindistrict.org, bitcoindistrict.org
- [x] Docker image: ghcr.io/tkhumush/bitcoindistrictrelay
- [x] Fresh git init with main branch, remote set to GitHub

## 📋 To Do

### 🔴 High Priority (Before Deployment)
- [ ] **Replace noteguard.toml whitelist** — Swap all nostrarabia community pubkeys for Bitcoin District community members. Remove irrelevant entries.
- [ ] **Generate fresh secrets** — Docker env vars, blossom secret key, any API keys need to be created fresh (never reuse nostrarabiarelay secrets)
- [ ] **Update SSL certificate paths** — All nginx/Caddy configs reference `nostrarabia.com` cert paths; update to `bitcoindistrict.org` Let's Encrypt paths
- [ ] **Update .github/workflows/deploy.yml** — Set correct deployment target server, SSH keys, and paths for the new infrastructure
- [ ] **Update README.md** — Rewrite from scratch to describe BitcoinDistrictRelay, its purpose, setup instructions, and community links

### 🟡 Medium Priority (Polish & Hardening)
- [ ] **Customize strfry.conf** — Update relay name, description, contact info, and any Arabia-specific settings
- [ ] **Customize blossom.yml** — Update blossom server config for new domain and storage settings
- [ ] **Customize nginx configs** — Update server_name, SSL paths, and any domain-specific headers in `nginx/conf.d/*.conf`
- [ ] **Review Caddyfile** — Ensure all reverse proxy rules, timeouts, and security headers are correct for new domains
- [ ] **Update docker-compose.yml** — Container names, volume names, network names should reflect Bitcoin District branding
- [ ] **Add .env.example** — Create a template env file documenting all required secrets without exposing actual values
- [ ] **Add HEALTH_CHECK.md** — Document how to verify the relay and blossom server are healthy after deployment

### 🟢 Nice to Have (Post-Launch)
- [ ] **Landing page** — Create a proper `index.html` for bitcoindistrict.org (currently just a placeholder)
- [ ] **NIP-11 relay info** — Customize the relay information document (name, description, supported NIPs, pubkey, contact)
- [ ] **Monitoring/alerting** — Set up health checks, uptime monitoring, and optional notifications
- [ ] **Backup strategy** — Document how to back up and restore the strfry database and blossom media storage
- [ ] **Contributing guide** — Add CONTRIBUTING.md if the project will be open to community contributions

---

## ⚠️ Important Notes
- **Do NOT push to GitHub** until Sir gives explicit approval
- **Do NOT reuse any secrets** from nostrarabiarelay — generate everything fresh
- The relay is whitelist-based (noteguard.toml) — manage pubkeys carefully
- Blossom server allows 100MB uploads — ensure adequate disk space on deployment server