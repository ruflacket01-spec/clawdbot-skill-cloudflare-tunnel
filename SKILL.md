---
name: cloudflare-tunnel
description: Deploy apps on subdomains via Cloudflare Tunnel with Zero Trust Access protection.
homepage: https://developers.cloudflare.com/cloudflare-one/connections/connect-networks/
metadata: {"clawdbot":{"emoji":"🚇"}}
---

# Cloudflare Tunnel Skill

Deploy local services to custom subdomains with Cloudflare Tunnel + Zero Trust Access.

## Prerequisites

- `cloudflared` CLI installed
- Cloudflare account with domain configured
- Tunnel credentials in `~/.cloudflared/`

## Setup (One-time per VPS)

```bash
# Install cloudflared
curl -L https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-amd64 -o /tmp/cloudflared
chmod +x /tmp/cloudflared && sudo mv /tmp/cloudflared /usr/local/bin/cloudflared

# Login to Cloudflare (opens browser)
cloudflared login

# Create a named tunnel
cloudflared tunnel create my-tunnel
# Note the Tunnel ID (e.g., a1b2c3d4-e5f6-...)
```

## Configuration

### Tunnel Config (`~/.cloudflared/config.yml`)

```yaml
tunnel: <TUNNEL_ID>
credentials-file: /home/ubuntu/.cloudflared/<TUNNEL_ID>.json

ingress:
  - hostname: app1.yourdomain.com
    service: http://localhost:3000
  - hostname: app2.yourdomain.com
    service: http://localhost:4000
  - service: http_status:404
```

## Deploy a New Service

### 1. Add hostname to config

Edit `~/.cloudflared/config.yml` and add your ingress rule.

### 2. Create DNS record

```bash
cloudflared tunnel route dns <TUNNEL_ID> <subdomain>.yourdomain.com
```

Or via API:
```bash
curl -X POST "https://api.cloudflare.com/client/v4/zones/<ZONE_ID>/dns_records" \
  -H "Authorization: Bearer <API_TOKEN>" \
  -H "Content-Type: application/json" \
  -d '{
    "type": "CNAME",
    "name": "<subdomain>",
    "content": "<TUNNEL_ID>.cfargotunnel.com",
    "proxied": true
  }'
```

### 3. Restart tunnel service

```bash
sudo systemctl restart cloudflared
```

### 4. (Optional) Add Zero Trust Access Policy

Via Cloudflare Zero Trust Dashboard (`https://one.dash.cloudflare.com`):
1. Access → Applications → Add Application → Self-hosted
2. Set domain: `<subdomain>.yourdomain.com`
3. Add policy: Allow specific emails

## Systemd Service

Create `/etc/systemd/system/cloudflared.service`:

```ini
[Unit]
Description=Cloudflare Tunnel
After=network.target

[Service]
Type=simple
User=ubuntu
ExecStart=/usr/local/bin/cloudflared tunnel --config /home/ubuntu/.cloudflared/config.yml run
Restart=always
RestartSec=5

[Install]
WantedBy=multi-user.target
```

Enable and start:
```bash
sudo systemctl daemon-reload
sudo systemctl enable cloudflared
sudo systemctl start cloudflared
```

## Quick Deploy Checklist

1. [ ] App running locally on a port
2. [ ] Add ingress rule to `~/.cloudflared/config.yml`
3. [ ] `cloudflared tunnel route dns <TUNNEL_ID> <subdomain>.yourdomain.com`
4. [ ] `sudo systemctl restart cloudflared`
5. [ ] (Optional) Add Access policy in Zero Trust dashboard
6. [ ] Test: `https://<subdomain>.yourdomain.com`

## Credentials Storage

Store API credentials for automation:
```
~/.config/cloudflare/api_token    # API Token (DNS Edit permission)
~/.config/cloudflare/zone_id      # Zone ID for your domain
~/.config/cloudflare/account_id   # Account ID for Access policies
```

## Troubleshooting

```bash
# Check tunnel status
sudo systemctl status cloudflared

# View logs
sudo journalctl -u cloudflared -f

# Test config
cloudflared tunnel --config ~/.cloudflared/config.yml ingress validate

# List tunnels
cloudflared tunnel list
```

## Multiple Services Example

```yaml
tunnel: a1b2c3d4-e5f6-7890-abcd-ef1234567890
credentials-file: /home/ubuntu/.cloudflared/a1b2c3d4-e5f6-7890-abcd-ef1234567890.json

ingress:
  # Kanban board
  - hostname: kanban.yourdomain.com
    service: http://localhost:4321
  
  # API server
  - hostname: api.yourdomain.com
    service: http://localhost:8080
  
  # Supabase Studio (local dev)
  - hostname: db.yourdomain.com
    service: http://localhost:54323
  
  # Catch-all (required)
  - service: http_status:404
```
