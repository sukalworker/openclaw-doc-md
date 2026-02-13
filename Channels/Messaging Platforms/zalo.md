# Zalo (Bot API)

Status: experimental. Direct messages only; groups coming soon per Zalo docs.

## Plugin required

Zalo ships as plugin, not bundled with core.

Install via CLI:
```bash
openclaw plugins install @openclaw/zalo
```

Or select Zalo during onboarding and confirm install prompt

Details: [Plugins](/tools/plugin)

## Quick setup (beginner)

- Install Zalo plugin: `openclaw plugins install @openclaw/zalo` or from source checkout
- Set the token:
  - Env: ZALO_BOT_TOKEN=...
  - Or config: channels.zalo.botToken: "..."
- Restart gateway (or finish onboarding)
- DM access pairing by default; approve pairing code on first contact

Minimal config:
```json
{
  "channels": {
    "zalo": {
      "enabled": true,
      "botToken": "12345689:abc-xyz",
      "dmPolicy": "pairing"
    }
  }
}
```

## What it is

Zalo is Vietnam-focused messaging app; Bot API lets Gateway run bot for 1:1 conversations.
Good fit for support or notifications where you want deterministic routing back to Zalo.

- Zalo Bot API channel owned by Gateway
- Deterministic routing: replies go back to Zalo; model never chooses channels
- DMs share agent main session
- Groups not yet supported (Zalo docs state "coming soon")

## Setup (fast path)

### 1) Create bot token (Zalo Bot Platform)

- Go to [https://bot.zaloplatforms.com](https://bot.zaloplatforms.com) and sign in
- Create new bot and configure settings
- Copy bot token (format: 12345689:abc-xyz)

### 2) Configure token (env or config)

Example:
```json
{
  "channels": {
    "zalo": {
      "enabled": true,
      "botToken": "12345689:abc-xyz",
      "dmPolicy": "pairing"
    }
  }
}
```

Env option: ZALO_BOT_TOKEN=... (works for default account only)

Multi-account support: use channels.zalo.accounts with per-account tokens and optional name.

- Restart gateway. Zalo starts when token resolved (env or config)
- DM access defaults to pairing. Approve code when bot first contacted.

## How it works (behavior)

- Inbound messages normalized into channel envelope with media placeholders
- Replies always route back to same Zalo chat
- Long-polling by default; webhook mode available with channels.zalo.webhookUrl

## Limits

- Outbound text chunked to 2000 characters (Zalo API limit)
- Media downloads/uploads capped by channels.zalo.mediaMaxMb (default 5)
- Streaming blocked by default (2000 char limit makes streaming less useful)

## Access control (DMs)

### DM access

- Default: channels.zalo.dmPolicy = "pairing" (unknown senders get pairing code; expire after 1 hour)
- Approve: `openclaw pairing list zalo` / `openclaw pairing approve zalo`
- Pairing is default token exchange (details: [Pairing](/channels/pairing))
- channels.zalo.allowFrom accepts numeric user IDs (no username lookup available)

## Long-polling vs webhook

- Default: long-polling (no public URL required)
- Webhook mode: set channels.zalo.webhookUrl and channels.zalo.webhookSecret

Webhook secret must be 8-256 characters.
- Webhook URL must use HTTPS
- Zalo sends events with X-Bot-Api-Secret-Token header for verification
- Gateway HTTP handles webhook requests at channels.zalo.webhookPath (defaults to webhook URL path)

Note: getUpdates (polling) and webhook mutually exclusive per Zalo API docs.

## Supported message types

### Receive
- Text messages: Full support with 2000 character chunking
- Image messages: Download and process inbound images; send via sendPhoto
- Stickers: Logged but not fully processed (no agent response)
- Unsupported types: Logged (e.g., messages from protected users)

## Capabilities

| Feature | Status |
|---------|--------|
| Direct messages | ✅ Supported |
| Groups | ❌ Coming soon (per Zalo docs) |
| Media (images) | ✅ Supported |
| Reactions | ❌ Not supported |
| Threads | ❌ Not supported |
| Polls | ❌ Not supported |
| Native commands | ❌ Not supported |
| Streaming | ⚠️ Blocked (2000 char limit) |

## Delivery targets (CLI/cron)

- Use chat id as target
- Example: `openclaw message send --channel zalo --target 123456789 --message "hi"`

## Troubleshooting

Bot doesn't respond:
- Check token valid: `openclaw channels status --probe`
- Verify sender approved (pairing or allowFrom)
- Check gateway logs: `openclaw logs --follow`

Webhook not receiving events:
- Ensure webhook URL uses HTTPS
- Verify secret token is 8-256 characters
- Confirm gateway HTTP endpoint reachable on configured path
- Check getUpdates polling not running (mutually exclusive)

## Configuration reference (Zalo)

Full configuration: [Configuration](/gateway/configuration)

Provider options:
- channels.zalo.enabled: enable/disable channel startup
- channels.zalo.botToken: bot token from Zalo Bot Platform
- channels.zalo.tokenFile: read token from file path
- channels.zalo.dmPolicy: pairing | allowlist | open | disabled (default: pairing)
- channels.zalo.allowFrom: DM allowlist (user IDs). open requires "*"
- channels.zalo.mediaMaxMb: inbound/outbound media cap (MB, default 5)
- channels.zalo.webhookUrl: enable webhook mode (HTTPS required)
- channels.zalo.webhookSecret: webhook secret (8-256 chars)
- channels.zalo.webhookPath: webhook path on gateway HTTP server
- channels.zalo.proxy: proxy URL for API requests

Multi-account options:
- channels.zalo.accounts..<id>.botToken: per-account token
- channels.zalo.accounts..<id>.tokenFile: per-account token file
- channels.zalo.accounts..<id>.name: display name
- channels.zalo.accounts..<id>.enabled: enable/disable account
- channels.zalo.accounts..<id>.dmPolicy: per-account DM policy
- channels.zalo.accounts..<id>.allowFrom: per-account allowlist
- channels.zalo.accounts..<id>.webhookUrl: per-account webhook URL
- channels.zalo.accounts..<id>.webhookSecret: per-account webhook secret
- channels.zalo.accounts..<id>.webhookPath: per-account webhook path
- channels.zalo.accounts..<id>.proxy: per-account proxy URL
