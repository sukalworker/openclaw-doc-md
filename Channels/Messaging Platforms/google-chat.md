# Google Chat

Status: ready for DMs + spaces via Google Chat API webhooks (HTTP only).

## Quick setup (beginner)

Create Google Cloud project and enable Google Chat API at [Google Chat API Credentials](https://console.cloud.google.com/apis/api/chat.googleapis.com/credentials).

Enable the API if not already enabled.

Create a Service Account:
- Press Create Credentials > Service Account
- Name it (e.g., openclaw-chat)
- Leave permissions blank (press Continue)
- Leave principals blank (press Done)

Create and download JSON Key:
- In service account list, click the one you created
- Go to Keys tab
- Click Add Key > Create new key
- Select JSON and press Create
- Store downloaded JSON on gateway host (e.g., ~/.openclaw/googlechat-service-account.json)

Create Google Chat app in [Google Cloud Console Chat Configuration](https://console.cloud.google.com/apis/api/chat.googleapis.com/hangouts-chat):

Fill in Application info:
- App name: (e.g. OpenClaw)
- Avatar URL: (e.g. https://openclaw.ai/logo.png)
- Description: (e.g. Personal AI Assistant)

Configure:
- Enable Interactive features
- Under Functionality, check Join spaces and group conversations
- Under Connection settings, select HTTP endpoint URL
- Under Triggers, select "Use a common HTTP endpoint URL for all triggers"
- Set to your gateway's public URL + /googlechat (run `openclaw status` to find)
- Under Visibility, check "Make this Chat app available to specific people and groups in"
- Enter your email (e.g user@example.com)
- Click Save

Enable app status:
- After saving, refresh page
- Look for App status section
- Change status to "Live - available to users"
- Click Save

Configure OpenClaw with service account path + webhook audience:
```
Env: GOOGLE_CHAT_SERVICE_ACCOUNT_FILE=/path/to/service-account.json
Or config: channels.googlechat.serviceAccountFile: "/path/to/service-account.json"
```

Set webhook audience type + value (matches your Chat app config).

Start gateway. Google Chat will POST to your webhook path.

## Add to Google Chat

Once gateway running and email added to visibility list:
- Go to [Google Chat](https://chat.google.com/)
- Click + (plus) next to Direct Messages
- In search bar, type the App name you configured
- Note: bot won't appear in "Marketplace" browse list (private app). Search by name.
- Select bot from results
- Click Add or Chat to start 1:1 conversation
- Send "Hello" to trigger assistant!

## Public URL (Webhook-only)

Google Chat webhooks require public HTTPS endpoint. For security, only expose /googlechat path to internet. Keep dashboard and sensitive endpoints on private network.

### Option A: Tailscale Funnel (Recommended)

Use Tailscale Serve for dashboard (private) and Funnel for public webhook path.

Check gateway bind address:
```bash
ss -tlnp | grep 18789
```

Expose dashboard to tailnet only (port 8443):
```bash
# If bound to localhost (127.0.0.1 or 0.0.0.0):
tailscale serve --bg --https 8443 http://127.0.0.1:18789

# If bound to Tailscale IP only (e.g., 100.106.161.80):
tailscale serve --bg --https 8443 http://100.106.161.80:18789
```

Expose only webhook path publicly:
```bash
# If bound to localhost:
tailscale funnel --bg --set-path /googlechat http://127.0.0.1:18789/googlechat

# If bound to Tailscale IP only:
tailscale funnel --bg --set-path /googlechat http://100.106.161.80:18789/googlechat
```

Authorize node for Funnel access if prompted.

Verify configuration:
```bash
tailscale serve status
tailscale funnel status
```

Your public webhook URL: `https://<node-name>.<tailnet>.ts.net/googlechat`
Your private dashboard: `https://<node-name>.<tailnet>.ts.net:8443/`
Use public URL (without :8443) in Google Chat app config.

This persists across reboots. To remove later: `tailscale funnel reset` and `tailscale serve reset`.

### Option B: Reverse Proxy (Caddy)

If using reverse proxy like Caddy, only proxy the specific path:
```
your-domain.com {
  reverse_proxy /googlechat* localhost:18789
}
```

Any request to your-domain.com/ ignored or 404, while your-domain.com/googlechat safely routed to OpenClaw.

### Option C: Cloudflare Tunnel

Configure tunnel's ingress rules:
- Path: /googlechat → http://localhost:18789/googlechat
- Default Rule: HTTP 404 (Not Found)

## How it works

- Google Chat sends webhook POSTs to gateway. Each request includes Authorization: Bearer header.
- OpenClaw verifies token against configured audienceType + audience:
  - audienceType: "app-url" → audience is your HTTPS webhook URL
  - audienceType: "project-number" → audience is Cloud project number
- Messages routed by space:
  - DMs use session key agent::googlechat:dm:
  - Spaces use session key agent::googlechat:group:
- DM access pairing by default. Unknown senders receive pairing code; approve with: `openclaw pairing approve googlechat <code>`
- Group spaces require @-mention by default. Use botUser if mention detection needs app's user name.

## Targets

Use these identifiers for delivery and allowlists:
- Direct messages: users/ or users/ (email addresses accepted)
- Spaces: spaces/

## Config highlights

```json
{
  "channels": {
    "googlechat": {
      "enabled": true,
      "serviceAccountFile": "/path/to/service-account.json",
      "audienceType": "app-url",
      "audience": "https://gateway.example.com/googlechat",
      "webhookPath": "/googlechat",
      "botUser": "users/1234567890",
      "dm": {
        "policy": "pairing",
        "allowFrom": ["users/1234567890", "user@example.com"]
      },
      "groupPolicy": "allowlist",
      "groups": {
        "spaces/AAAA": {
          "allow": true,
          "requireMention": true,
          "users": ["users/1234567890"],
          "systemPrompt": "Short answers only."
        }
      },
      "actions": { "reactions": true },
      "typingIndicator": "message",
      "mediaMaxMb": 20
    }
  }
}
```

Notes:
- Service account credentials can also be passed inline with serviceAccount (JSON string)
- Default webhook path is /googlechat if webhookPath isn't set
- Reactions available via reactions tool when actions.reactions enabled
- typingIndicator supports none, message (default), reaction (reaction requires user OAuth)
- Attachments downloaded through Chat API and stored in media pipeline (size capped by mediaMaxMb)

## Troubleshooting

### 405 Method Not Allowed

If Google Cloud Logs Explorer shows status code 405, webhook handler isn't registered.

Common causes:
- Channel not configured: channels.googlechat section missing. Verify: `openclaw config get channels.googlechat`
  If "Config path not found", add configuration (see Config highlights above)
- Plugin not enabled: Check: `openclaw plugins list | grep googlechat`
  If shows "disabled", add plugins.entries.googlechat.enabled: true to config
- Gateway not restarted: After adding config, restart: `openclaw gateway restart`

Verify channel running: `openclaw channels status` (should show: Google Chat default: enabled, configured, ...)

### Other issues

- Check `openclaw channels status --probe` for auth errors or missing audience config
- If no messages arrive, confirm Chat app webhook URL + event subscriptions
- If mention gating blocks replies, set botUser to app's user resource name and verify requireMention
- Use `openclaw logs --follow` while sending test message to see if requests reach gateway

Related docs:
- [Gateway configuration](/gateway/configuration)
- [Security](/gateway/security)
- [Reactions](/tools/reactions)
