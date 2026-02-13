# BlueBubbles (macOS REST)

Status: bundled plugin that talks to BlueBubbles macOS server over HTTP. Recommended for iMessage integration due to richer API and easier setup compared to legacy imsg channel.

## Overview

- Runs on macOS via BlueBubbles helper app ([bluebubbles.app](https://bluebubbles.app))
- Recommended/tested: macOS Sequoia (15). macOS Tahoe (26) works; edit is currently broken on Tahoe, group icon updates may report success but not sync
- OpenClaw talks to it through REST API (GET /api/v1/ping, POST /message/text, POST /chat/:id/*)
- Incoming messages arrive via webhooks; outgoing replies, typing, read receipts, tapbacks are REST calls
- Attachments and stickers ingested as inbound media
- Pairing/allowlist same as other channels with channels.bluebubbles.allowFrom + pairing codes
- Reactions surfaced as system events
- Advanced features: edit, unsend, reply threading, message effects, group management

## Quick start

- Install BlueBubbles server on your Mac ([bluebubbles.app/install](https://bluebubbles.app/install))
- In BlueBubbles config, enable web API and set password
- Run `openclaw onboard` and select BlueBubbles, or configure manually:

```json
{
  "channels": {
    "bluebubbles": {
      "enabled": true,
      "serverUrl": "http://192.168.1.100:1234",
      "password": "example-password",
      "webhookPath": "/bluebubbles-webhook"
    }
  }
}
```

- Point BlueBubbles webhooks to your gateway (example: https://your-gateway-host:3000/bluebubbles-webhook?password=)
- Start gateway; it will register webhook handler and start pairing

## Keeping Messages.app alive (VM / headless setups)

Some macOS VM / always-on setups can end up with Messages.app going "idle" (incoming events stop until app is opened). Workaround is to poke Messages every 5 minutes using AppleScript + LaunchAgent.

### 1) Save the AppleScript

Save as ~/Scripts/poke-messages.scpt:
```applescript
try
  tell application "Messages"
    if not running then
      launch
    end if
    -- Touch the scripting interface to keep the process responsive.
    set _chatCount to (count of chats)
  end tell
on error
  -- Ignore transient failures (first-run prompts, locked session, etc).
end try
```

### 2) Install a LaunchAgent

Save as ~/Library/LaunchAgents/com.user.poke-messages.plist:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
  <dict>
    <key>Label</key>
    <string>com.user.poke-messages</string>
    <key>ProgramArguments</key>
    <array>
      <string>/bin/bash</string>
      <string>-lc</string>
      <string>/usr/bin/osascript "$HOME/Scripts/poke-messages.scpt"</string>
    </array>
    <key>RunAtLoad</key>
    <true/>
    <key>StartInterval</key>
    <integer>300</integer>
    <key>StandardOutPath</key>
    <string>/tmp/poke-messages.log</string>
    <key>StandardErrorPath</key>
    <string>/tmp/poke-messages.err</string>
  </dict>
</plist>
```

Load it:
```bash
launchctl unload ~/Library/LaunchAgents/com.user.poke-messages.plist 2>/dev/null || true
launchctl load ~/Library/LaunchAgents/com.user.poke-messages.plist
```

## Onboarding

BlueBubbles available in interactive setup:
```bash
openclaw onboard
```

Prompts for:
- Server URL (required): BlueBubbles server address (e.g., http://192.168.1.100:1234)
- Password (required): API password from BlueBubbles Server settings
- Webhook path (optional): Defaults to /bluebubbles-webhook
- DM policy: pairing, allowlist, open, or disabled
- Allow list: Phone numbers, emails, or chat targets

Add via CLI:
```bash
openclaw channels add bluebubbles --http-url http://192.168.1.100:1234 --password <password>
```

## Access control (DMs + groups)

### DMs

- Default: channels.bluebubbles.dmPolicy = "pairing"
- Unknown senders get pairing code; ignored until approved (1 hour expiry)
- Approve: `openclaw pairing list bluebubbles` / `openclaw pairing approve bluebubbles`
- Pairing is default token exchange (see [Pairing](/channels/pairing))

### Groups

- channels.bluebubbles.groupPolicy = open | allowlist | disabled (default: allowlist)
- channels.bluebubbles.groupAllowFrom controls who can trigger in groups when allowlist set

### Mention gating (groups)

BlueBubbles supports mention gating for group chats:
- Uses agents.list[].groupChat.mentionPatterns (or messages.groupChat.mentionPatterns)
- When requireMention enabled for group, agent only responds when mentioned
- Control commands from authorized senders bypass mention gating

Per-group configuration:
```json
{
  "channels": {
    "bluebubbles": {
      "groupPolicy": "allowlist",
      "groupAllowFrom": ["+15555550123"],
      "groups": {
        "*": { "requireMention": true },
        "iMessage;-;chat123": { "requireMention": false }
      }
    }
  }
}
```

### Command gating

- Control commands (e.g., /config, /model) require authorization
- Uses allowFrom and groupAllowFrom
- Authorized senders can run control commands even without mentioning in groups

## Typing + read receipts

- Typing indicators: sent automatically before/during response generation
- Read receipts: controlled by channels.bluebubbles.sendReadReceipts (default: true)
- Typing: OpenClaw sends typing start; BlueBubbles clears automatically on send or timeout

```json
{
  "channels": {
    "bluebubbles": {
      "sendReadReceipts": false
    }
  }
}
```

## Advanced actions

BlueBubbles supports advanced message actions when enabled:

```json
{
  "channels": {
    "bluebubbles": {
      "actions": {
        "reactions": true,          // tapbacks (default: true)
        "edit": true,                // edit sent messages (macOS 13+, broken on macOS 26 Tahoe)
        "unsend": true,              // unsend messages (macOS 13+)
        "reply": true,               // reply threading by message GUID
        "sendWithEffect": true,      // message effects (slam, loud, etc.)
        "renameGroup": true,         // rename group chats
        "setGroupIcon": true,        // set group chat icon/photo (flaky on macOS 26 Tahoe)
        "addParticipant": true,      // add participants to groups
        "removeParticipant": true,   // remove participants
        "leaveGroup": true,          // leave group chats
        "sendAttachment": true       // send attachments/media
      }
    }
  }
}
```

Available actions:
- react: Add/remove tapback reactions (messageId, emoji, remove)
- edit: Edit sent message (messageId, text)
- unsend: Unsend message (messageId)
- reply: Reply to specific message (messageId, text, to)
- sendWithEffect: Send with iMessage effect (text, to, effectId)
- renameGroup: Rename group chat (chatGuid, displayName)
- setGroupIcon: Set group chat icon/photo (chatGuid, media) — flaky on macOS 26 Tahoe
- addParticipant: Add someone to group (chatGuid, address)
- removeParticipant: Remove someone from group (chatGuid, address)
- leaveGroup: Leave group chat (chatGuid)
- sendAttachment: Send media/files (to, buffer, filename, asVoice)

Voice memos: set asVoice: true with MP3 or CAF audio to send as iMessage voice message. BlueBubbles converts MP3 → CAF.

### Message IDs (short vs full)

OpenClaw may surface short message IDs (e.g., 1, 2) to save tokens.
- MessageSid / ReplyToId: can be short IDs
- MessageSidFull / ReplyToIdFull: contain provider full IDs
- Short IDs in-memory; can expire on restart or cache eviction
- Actions accept short or full, but short IDs error if no longer available

Use full IDs for durable automations:
- Templates: {{MessageSidFull}}, {{ReplyToIdFull}}
- Context: MessageSidFull / ReplyToIdFull in inbound payloads

## Block streaming

Control whether responses sent as single message or streamed in blocks:
```json
{
  "channels": {
    "bluebubbles": {
      "blockStreaming": true
    }
  }
}
```

- Inbound attachments downloaded and stored in media cache
- Media cap via channels.bluebubbles.mediaMaxMb (default: 8 MB)
- Outbound text chunked to channels.bluebubbles.textChunkLimit (default: 4000 chars)

## Configuration reference

Full configuration: [Configuration](/gateway/configuration)

Provider options:
- channels.bluebubbles.enabled: Enable/disable channel
- channels.bluebubbles.serverUrl: BlueBubbles REST API base URL
- channels.bluebubbles.password: API password
- channels.bluebubbles.webhookPath: Webhook endpoint path (default: /bluebubbles-webhook)
- channels.bluebubbles.dmPolicy: pairing | allowlist | open | disabled (default: pairing)
- channels.bluebubbles.allowFrom: DM allowlist
- channels.bluebubbles.groupPolicy: open | allowlist | disabled (default: allowlist)
- channels.bluebubbles.groupAllowFrom: Group sender allowlist
- channels.bluebubbles.groups: Per-group config
- channels.bluebubbles.sendReadReceipts: Send read receipts (default: true)
- channels.bluebubbles.blockStreaming: Enable block streaming (default: false)
- channels.bluebubbles.textChunkLimit: Outbound chunk size (default: 4000)
- channels.bluebubbles.chunkMode: length (default) or newline
- channels.bluebubbles.mediaMaxMb: Inbound media cap (default: 8)
- channels.bluebubbles.historyLimit: Max group messages for context (0 disables)
- channels.bluebubbles.dmHistoryLimit: DM history limit
- channels.bluebubbles.actions: Enable/disable specific actions
- channels.bluebubbles.accounts: Multi-account configuration

Related global options:
- agents.list[].groupChat.mentionPatterns (or messages.groupChat.mentionPatterns)
- messages.responsePrefix

## Addressing / delivery targets

Prefer chat_guid for stable routing:
- chat_guid:iMessage;-;+15555550123 (preferred for groups)
- chat_id:123
- chat_identifier:...
- Direct handles: +15555550123, user@example.com

If direct handle doesn't have existing DM chat, OpenClaw creates one via POST /api/v1/chat/new (requires BlueBubbles Private API enabled).

## Security

- Webhook requests authenticated by comparing guid/password query params/headers against channels.bluebubbles.password
- Localhost requests also accepted
- Keep API password and webhook endpoint secret (like credentials)
- Localhost trust means same-host reverse proxy can bypass password — if proxying, require auth at proxy and configure gateway.trustedProxies
- Enable HTTPS + firewall rules on BlueBubbles if exposing outside LAN

## Troubleshooting

- Typing/read events stop: check BlueBubbles webhook logs, verify gateway path matches channels.bluebubbles.webhookPath
- Pairing codes expire: 1 hour; use `openclaw pairing list bluebubbles` and re-approve
- Reactions need BlueBubbles private API (POST /api/v1/message/react)
- Edit/unsend require macOS 13+. On macOS 26 (Tahoe), edit broken due to private API changes
- Group icon flaky on macOS 26 (Tahoe): API returns success but icon doesn't sync
- OpenClaw auto-hides known-broken actions based on macOS version; if edit still appears on macOS 26, disable manually
- Status/health: `openclaw status --all` or `openclaw status --deep`

For general channel workflow: [Channels](/channels) and [Plugins](/tools/plugin) guide.
