# LINE (plugin)

LINE connects to OpenClaw via LINE Messaging API. Plugin runs as webhook receiver on gateway and uses channel access token + channel secret for authentication.

Status: supported via plugin. Direct messages, group chats, media, locations, Flex messages, template messages, and quick replies supported. Reactions and threads not supported.

## Plugin required

Install LINE plugin:
```bash
openclaw plugins install @openclaw/line
```

Local checkout (git repo):
```bash
openclaw plugins install ./extensions/line
```

## Setup

- Create LINE Developers account and open Console: [https://developers.line.biz/console/](https://developers.line.biz/console/)
- Create (or pick) a Provider and add Messaging API channel
- Copy the Channel access token and Channel secret from channel settings
- Enable "Use webhook" in Messaging API settings
- Set webhook URL to your gateway endpoint (HTTPS required): `https://gateway-host/line/webhook`

Gateway responds to LINE's webhook verification (GET) and inbound events (POST).
If you need custom path, set channels.line.webhookPath or channels.line.accounts.<id>.webhookPath and update URL accordingly.

## Configure

Minimal config:
```json
{
  "channels": {
    "line": {
      "enabled": true,
      "channelAccessToken": "LINE_CHANNEL_ACCESS_TOKEN",
      "channelSecret": "LINE_CHANNEL_SECRET",
      "dmPolicy": "pairing"
    }
  }
}
```

Env vars (default account only):
- LINE_CHANNEL_ACCESS_TOKEN
- LINE_CHANNEL_SECRET

Token/secret files:
```json
{
  "channels": {
    "line": {
      "tokenFile": "/path/to/line-token.txt",
      "secretFile": "/path/to/line-secret.txt"
    }
  }
}
```

Multiple accounts:
```json
{
  "channels": {
    "line": {
      "accounts": {
        "marketing": {
          "channelAccessToken": "...",
          "channelSecret": "...",
          "webhookPath": "/line/marketing"
        }
      }
    }
  }
}
```

## Access control

Direct messages default to pairing. Unknown senders get pairing code and messages ignored until approved.

```bash
openclaw pairing list line
openclaw pairing approve line <CODE>
```

Allowlists and policies:
- channels.line.dmPolicy: pairing | allowlist | open | disabled
- channels.line.allowFrom: allowlisted LINE user IDs for DMs
- channels.line.groupPolicy: allowlist | open | disabled
- channels.line.groupAllowFrom: allowlisted LINE user IDs for groups
- Per-group overrides: channels.line.groups..<id>.allowFrom

LINE IDs are case-sensitive. Valid IDs:
- User: U + 32 hex chars
- Group: C + 32 hex chars
- Room: R + 32 hex chars

## Message behavior

- Text chunked at 5000 characters
- Markdown formatting stripped; code blocks and tables converted to Flex cards when possible
- Streaming responses buffered; LINE receives full chunks with loading animation while agent works
- Media downloads capped by channels.line.mediaMaxMb (default 10)

## Channel data (rich messages)

Use channelData.line to send quick replies, locations, Flex cards, or template messages.

```json
{
  "text": "Here you go",
  "channelData": {
    "line": {
      "quickReplies": ["Status", "Help"],
      "location": {
        "title": "Office",
        "address": "123 Main St",
        "latitude": 35.681236,
        "longitude": 139.767125
      },
      "flexMessage": {
        "altText": "Status card",
        "contents": { /* Flex payload */ }
      },
      "templateMessage": {
        "type": "confirm",
        "text": "Proceed?",
        "confirmLabel": "Yes",
        "confirmData": "yes",
        "cancelLabel": "No",
        "cancelData": "no"
      }
    }
  }
}
```

Plugin also ships /card command for Flex message presets:
```bash
/card info "Welcome" "Thanks for joining!"
```

## Troubleshooting

- Webhook verification fails: ensure webhook URL is HTTPS and channelSecret matches LINE console
- No inbound events: confirm webhook path matches channels.line.webhookPath and gateway reachable from LINE
- Media download errors: raise channels.line.mediaMaxMb if media exceeds default limit
