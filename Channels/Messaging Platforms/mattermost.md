# Mattermost (plugin)

Status: supported via plugin (bot token + WebSocket events). Channels, groups, and DMs supported.
Mattermost is self-hostable team messaging platform ([mattermost.com](https://mattermost.com)).

## Plugin required

Mattermost ships as plugin, not bundled with core.

Install via CLI (npm registry):
```bash
openclaw plugins install @openclaw/mattermost
```

Local checkout (git repo):
```bash
openclaw plugins install ./extensions/mattermost
```

If you choose Mattermost during configure/onboarding and git checkout detected, OpenClaw will offer local install path automatically.

Details: [Plugins](/tools/plugin)

## Quick setup

- Install Mattermost plugin
- Create Mattermost bot account and copy bot token
- Copy Mattermost base URL (e.g., https://chat.example.com)
- Configure OpenClaw and start gateway

Minimal config:
```json
{
  "channels": {
    "mattermost": {
      "enabled": true,
      "botToken": "mm-token",
      "baseUrl": "https://chat.example.com",
      "dmPolicy": "pairing"
    }
  }
}
```

## Environment variables (default account)

Set on gateway host if you prefer env vars:
- MATTERMOST_BOT_TOKEN=...
- MATTERMOST_URL=https://chat.example.com

Env vars apply only to default account. Other accounts must use config values.

## Chat modes

Mattermost responds to DMs automatically. Channel behavior controlled by chatmode:
- **oncall** (default): respond only when @mentioned in channels
- **onmessage**: respond to every channel message
- **onchar**: respond when message starts with trigger prefix

Config example:
```json
{
  "channels": {
    "mattermost": {
      "chatmode": "onchar",
      "oncharPrefixes": [">", "!"]
    }
  }
}
```

Notes:
- onchar still responds to explicit @mentions
- channels.mattermost.requireMention honored for legacy configs; chatmode preferred

## Access control (DMs)

- Default: channels.mattermost.dmPolicy = "pairing" (unknown senders get pairing code)
- Approve: `openclaw pairing list mattermost` / `openclaw pairing approve mattermost`
- Public DMs: channels.mattermost.dmPolicy="open" plus channels.mattermost.allowFrom=["*"]

## Channels (groups)

- Default: channels.mattermost.groupPolicy = "allowlist" (mention-gated)
- Allowlist senders with channels.mattermost.groupAllowFrom (user IDs or @username)
- Open channels: channels.mattermost.groupPolicy="open" (mention-gated)

## Targets for outbound delivery

Use these target formats with `openclaw message send` or cron/webhooks:
- channel: for a channel
- user: for a DM
- @username for a DM (resolved via Mattermost API)

Bare IDs treated as channels.

## Multi-account

Mattermost supports multiple accounts under channels.mattermost.accounts:
```json
{
  "channels": {
    "mattermost": {
      "accounts": {
        "default": {
          "name": "Primary",
          "botToken": "mm-token",
          "baseUrl": "https://chat.example.com"
        },
        "alerts": {
          "name": "Alerts",
          "botToken": "mm-token-2",
          "baseUrl": "https://alerts.example.com"
        }
      }
    }
  }
}
```

## Troubleshooting

- No replies in channels: ensure bot is in channel and mention it (oncall), use trigger prefix (onchar), or set chatmode: "onmessage"
- Auth errors: check bot token, base URL, and whether account is enabled
- Multi-account issues: env vars only apply to default account
