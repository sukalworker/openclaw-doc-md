# Nextcloud Talk (plugin)

Status: supported via plugin (webhook bot). Direct messages, rooms, reactions, and markdown messages supported.

## Plugin required

Nextcloud Talk ships as plugin, not bundled with core.

Install via CLI (npm registry):
```bash
openclaw plugins install @openclaw/nextcloud-talk
```

Local checkout (git repo):
```bash
openclaw plugins install ./extensions/nextcloud-talk
```

If you choose Nextcloud Talk during configure/onboarding and git checkout detected, OpenClaw will offer local install path automatically.

Details: [Plugins](/tools/plugin)

## Quick setup (beginner)

- Install Nextcloud Talk plugin
- On your Nextcloud server, create bot: `./occ talk:bot:install "OpenClaw" "" "" --feature reaction`
- Enable bot in target room settings
- Configure OpenClaw:
  - Config: channels.nextcloud-talk.baseUrl + channels.nextcloud-talk.botSecret
  - Or env: NEXTCLOUD_TALK_BOT_SECRET (default account only)
- Restart gateway (or finish onboarding)

Minimal config:
```json
{
  "channels": {
    "nextcloud-talk": {
      "enabled": true,
      "baseUrl": "https://cloud.example.com",
      "botSecret": "shared-secret",
      "dmPolicy": "pairing"
    }
  }
}
```

## Notes

- Bots cannot initiate DMs. User must message bot first.
- Webhook URL must be reachable by Gateway; set webhookPublicUrl if behind proxy
- Media uploads not supported by bot API; media sent as URLs
- Webhook payload doesn't distinguish DMs vs rooms; set apiUser + apiPassword to enable room-type lookups (otherwise DMs treated as rooms)

## Access control (DMs)

- Default: channels.nextcloud-talk.dmPolicy = "pairing" (unknown senders get pairing code)
- Approve: `openclaw pairing list nextcloud-talk` / `openclaw pairing approve nextcloud-talk`
- Public DMs: channels.nextcloud-talk.dmPolicy="open" plus channels.nextcloud-talk.allowFrom=["*"]
- allowFrom matches Nextcloud user IDs only; display names ignored

## Rooms (groups)

- Default: channels.nextcloud-talk.groupPolicy = "allowlist" (mention-gated)
- Allowlist rooms with channels.nextcloud-talk.rooms:

```json
{
  "channels": {
    "nextcloud-talk": {
      "rooms": {
        "room-token": { "requireMention": true }
      }
    }
  }
}
```

- To allow no rooms, keep allowlist empty or set channels.nextcloud-talk.groupPolicy="disabled"

## Capabilities

| Feature | Status |
|---------|--------|
| Direct messages | Supported |
| Rooms | Supported |
| Threads | Not supported |
| Media | URL-only |
| Reactions | Supported |
| Native commands | Not supported |

## Configuration reference (Nextcloud Talk)

Full configuration: [Configuration](/gateway/configuration)

Provider options:
- channels.nextcloud-talk.enabled: enable/disable channel startup
- channels.nextcloud-talk.baseUrl: Nextcloud instance URL
- channels.nextcloud-talk.botSecret: bot shared secret
- channels.nextcloud-talk.botSecretFile: secret file path
- channels.nextcloud-talk.apiUser: API user for room lookups (DM detection)
- channels.nextcloud-talk.apiPassword: API/app password for room lookups
- channels.nextcloud-talk.apiPasswordFile: API password file path
- channels.nextcloud-talk.webhookPort: webhook listener port (default: 8788)
- channels.nextcloud-talk.webhookHost: webhook host (default: 0.0.0.0)
- channels.nextcloud-talk.webhookPath: webhook path (default: /nextcloud-talk-webhook)
- channels.nextcloud-talk.webhookPublicUrl: externally reachable webhook URL
- channels.nextcloud-talk.dmPolicy: pairing | allowlist | open | disabled
- channels.nextcloud-talk.allowFrom: DM allowlist (user IDs). open requires "*"
- channels.nextcloud-talk.groupPolicy: allowlist | open | disabled
- channels.nextcloud-talk.groupAllowFrom: group allowlist (user IDs)
- channels.nextcloud-talk.rooms: per-room settings and allowlist
- channels.nextcloud-talk.historyLimit: group history limit (0 disables)
- channels.nextcloud-talk.dmHistoryLimit: DM history limit (0 disables)
- channels.nextcloud-talk.dms: per-DM overrides (historyLimit)
- channels.nextcloud-talk.textChunkLimit: outbound text chunk size (chars)
- channels.nextcloud-talk.chunkMode: length (default) or newline
- channels.nextcloud-talk.blockStreaming: disable block streaming
- channels.nextcloud-talk.blockStreamingCoalesce: block streaming coalesce tuning
- channels.nextcloud-talk.mediaMaxMb: inbound media cap (MB)
