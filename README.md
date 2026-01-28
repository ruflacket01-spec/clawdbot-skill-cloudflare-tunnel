# 🚇 Cloudflare Tunnel Skill for Clawdbot

Deploy local services to custom subdomains via Cloudflare Tunnel with optional Zero Trust Access protection.

## Features

- 🚇 Expose local services without opening firewall ports
- 🔒 Zero Trust Access for authentication (email/Google/GitHub login)
- 🌐 Custom subdomains (e.g., `app.yourdomain.com`)
- 🔄 Systemd service for auto-restart
- 🆓 Free tier available (50 users on Access)

## Use Cases

- Deploy a local dev server for client demos
- Self-host internal tools with auth protection
- Expose Supabase Studio for remote access
- Share local apps without port forwarding

## Installation

Copy `SKILL.md` to your Clawdbot skills directory:

```bash
mkdir -p ~/clawd/skills/cloudflare-tunnel
cp SKILL.md ~/clawd/skills/cloudflare-tunnel/
```

## Usage with Clawdbot

Once configured, ask Alfred:

> "Deploy my app on port 3000 to myapp.mydomain.com"

Alfred will:
1. Update the tunnel config
2. Create the DNS CNAME record
3. Restart the tunnel service
4. (Optional) Add Access protection

## Requirements

- Cloudflare account with a domain
- `cloudflared` CLI installed and authenticated
- Tunnel created with credentials stored

## Quick Start

```bash
# Install cloudflared
curl -L https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-amd64 -o /tmp/cloudflared
chmod +x /tmp/cloudflared && sudo mv /tmp/cloudflared /usr/local/bin/cloudflared

# Login (opens browser)
cloudflared login

# Create tunnel
cloudflared tunnel create my-tunnel

# Follow SKILL.md for config setup
```

## License

MIT

## Credits

Created for [Clawdbot](https://github.com/clawdbot/clawdbot) - Your AI-powered personal assistant.
