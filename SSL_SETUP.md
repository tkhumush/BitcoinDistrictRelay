# Manual SSL Setup Guide

This guide shows you how to manually set up SSL certificates with Let's Encrypt after your services are running.

## Prerequisites

1. Services are deployed and running: `docker compose ps`
2. DNS records point to your server (172.105.154.238):
   - `bitcoindistrict.org`
   - `relay.bitcoindistrict.org`
   - `cherry.bitcoindistrict.org`
3. Port 80 is accessible from the internet

## Step 1: Generate SSL Certificates

SSH into your server and run:

```bash
cd /home/bitcoindistrict/nostr-relay

# Obtain certificates for all domains
docker compose run --rm certbot certonly \
  --webroot \
  --webroot-path=/var/www/certbot \
  --email admin@bitcoindistrict.org \
  --agree-tos \
  --no-eff-email \
  -d bitcoindistrict.org \
  -d relay.bitcoindistrict.org \
  -d cherry.bitcoindistrict.org
```

This should complete in under a minute. If it fails, check:
- DNS records are correctly set
- Port 80 is open and accessible
- nginx is running: `docker compose ps nginx`

## Step 2: Enable HTTPS in Nginx

Edit each nginx config file to enable HTTPS:

```bash
cd /home/bitcoindistrict/nostr-relay/nginx/conf.d

# Edit strfry.conf
nano strfry.conf
# Uncomment the HTTPS server block (lines starting with #)
# Comment out or remove the HTTP server block

# Edit media.conf
nano media.conf
# Uncomment the HTTPS server block
# Comment out or remove the HTTP server block

# Edit default.conf
nano default.conf
# Uncomment the HTTPS server block
# Comment out or remove the HTTP server block
```

**Tip:** Look for sections marked with `# HTTPS Configuration (commented out - uncomment after SSL setup)`

## Step 3: Reload Nginx

```bash
cd /home/bitcoindistrict/nostr-relay

# Test nginx configuration
docker compose exec nginx nginx -t

# If test passes, reload nginx
docker compose exec nginx nginx -s reload
```

## Step 4: Verify SSL

Test your domains:

```bash
# Test main domain
curl -I https://bitcoindistrict.org

# Test relay
curl -I https://relay.bitcoindistrict.org

# Test media server
curl -I https://cherry.bitcoindistrict.org
```

All should return HTTPS responses with valid certificates!

## Certificate Renewal

Certificates auto-renew every 12 hours via the certbot container. Check renewal status:

```bash
# List certificates
docker compose run --rm certbot certificates

# Force renewal (for testing)
docker compose run --rm certbot renew --force-renewal
```

## Troubleshooting

**Certificate generation fails:**
```bash
# Check nginx logs
docker compose logs nginx

# Check certbot logs
docker compose logs certbot

# Verify DNS
nslookup bitcoindistrict.org
nslookup relay.bitcoindistrict.org
nslookup cherry.bitcoindistrict.org
```

**502 Bad Gateway after enabling HTTPS:**
```bash
# Check if backend services are running
docker compose ps

# Check backend logs
docker compose logs nostr-relay
docker compose logs blossom-server
```

**Need to start over:**
```bash
# Remove certificates
docker compose run --rm certbot delete --cert-name bitcoindistrict.org

# Start from Step 1 again
```
