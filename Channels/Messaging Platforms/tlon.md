# Tlon (plugin)

Tlon is decentralized messenger built on Urbit. OpenClaw connects to your Urbit ship and can respond to DMs and group chat messages. Group replies require @ mention by default and can be further restricted via allowlists.

Status: supported via plugin. DMs, group mentions, thread replies, and text-only media fallback (URL appended to caption) supported. Reactions, polls, and native media uploads not supported.

## Plugin required

Tlon ships as plugin, not bundled with core.

Install via CLI (npm registry):
```bash
openclaw plugins install @openclaw/tlon
```

Local checkout (git repo):
```bash
openclaw plugins install ./extensions/tlon
```

Details: [Plugins](/tools/plugin)

## Setup

- Install Tlon plugin
- Gather your ship URL and login code
- Configure channels.tlon
- Restart gateway
- DM bot or mention it in group channel

Minimal config (single account):
```json
{
  "channels": {
    "tlon": {
      "enabled": true,
      "ship": "~sampel-palnet",
      "url": "https://your-ship-host",
      "code": "lidlut-tabwed-pillex-ridrup"
    }
  }
}
```

## Group channels

Auto-discovery enabled by default. Can also pin channels manually:
```json
{
  "channels": {
    "tlon": {
      "groupChannels": ["chat/~host-ship/general", "chat/~host-ship/support"]
    }
  }
}
```

Disable auto-discovery:
```json
{
  "channels": {
    "tlon": {
      "autoDiscoverChannels": false
    }
  }
}
```

## Access control

DM allowlist (empty = allow all):
```json
{
  "channels": {
    "tlon": {
      "dmAllowlist": ["~zod", "~nec"]
    }
  }
}
```

Group authorization (restricted by default):
```json
{
  "channels": {
    "tlon": {
      "defaultAuthorizedShips": ["~zod"],
      "authorization": {
        "channelRules": {
          "chat/~host-ship/general": {
            "mode": "restricted",
            "allowedShips": ["~zod", "~nec"]
          },
          "chat/~host-ship/announcements": {
            "mode": "open"
          }
        }
      }
    }
  }
}
```

## Delivery targets (CLI/cron)

Use these with `openclaw message send` or cron delivery:
- DM: ~sampel-palnet or dm/~sampel-palnet
- Group: chat/~host-ship/channel or group:~host-ship/channel

## Notes

- Group replies require mention (e.g. ~your-bot-ship) to respond
- Thread replies: if inbound message in thread, OpenClaw replies in-thread
- Media: sendMedia falls back to text + URL (no native upload)
