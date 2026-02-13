# Signal

Status: external CLI integration. Gateway talks to signal-cli over HTTP JSON-RPC + SSE.

## Quick setup (beginner)

- Use a separate Signal number for the bot (recommended)
- Install signal-cli (Java required)
- Link the bot device: `signal-cli link -n "OpenClaw"` then scan QR
- Configure OpenClaw and start gateway

Minimal config:
```json
{
  "channels": {
    "signal": {
      "enabled": true,
      "account": "+15551234567",
      "cliPath": "signal-cli",
      "dmPolicy": "pairing",
      "allowFrom": ["+15557654321"]
    }
  }
}
```

## What it is

- Signal channel via signal-cli (not embedded libsignal)
- Deterministic routing: replies always go back to Signal
- DMs share agent main session; groups isolated (agent::signal:group:)

## Config writes

Signal allows config updates via /config set|unset (requires commands.config: true).
Disable with channels.signal.configWrites: false.

## The number model (important)

- Gateway connects to Signal device (signal-cli account)
- If using personal Signal account, ignores own messages (loop protection)
- For "I text the bot and it replies", use separate bot number

## Setup (fast path)

- Install signal-cli (Java required)
- Link bot account: `signal-cli link -n "OpenClaw"`, scan QR in Signal
- Configure Signal and start gateway

Multi-account support: channels.signal.accounts with per-account config and optional name.

## External daemon mode (httpUrl)

Run signal-cli separately and point OpenClaw at it:
```json
{
  "channels": {
    "signal": {
      "httpUrl": "http://127.0.0.1:8080",
      "autoStart": false
    }
  }
}
```

Skips auto-spawn and startup wait. For slow starts, set channels.signal.startupTimeoutMs.

## Access control (DMs + groups)

### DMs

- Default: channels.signal.dmPolicy = "pairing"
- Unknown senders receive pairing code; ignored until approved (1 hour expiry)
- Approve: `openclaw pairing list signal` / `openclaw pairing approve signal`
- UUID-only senders stored as uuid: in channels.signal.allowFrom

### Groups

- channels.signal.groupPolicy = open | allowlist | disabled
- channels.signal.groupAllowFrom controls who can trigger when allowlist set

## How it works (behavior)

- signal-cli runs as daemon; gateway reads events via SSE
- Inbound messages normalized into channel envelope
- Replies always route back to same number or group
- Outbound text chunked to channels.signal.textChunkLimit (default 4000)
- Optional newline chunking: set channels.signal.chunkMode="newline" for paragraph boundaries
- Attachments supported (base64 fetched from signal-cli)
- Media cap: channels.signal.mediaMaxMb (default 8)
- Use channels.signal.ignoreAttachments to skip media downloads
- Group history uses channels.signal.historyLimit (default 50, set 0 to disable)

## Typing + read receipts

- Typing indicators: OpenClaw sends typing signals and refreshes while generating reply
- Read receipts: when channels.signal.sendReadReceipts true, forwards read receipts for allowed DMs
- signal-cli does not expose read receipts for groups

## Reactions (message tool)

Use message action=react with channel=signal.
- Targets: sender E.164 or UUID (use uuid: from pairing output; bare UUID works)
- messageId: Signal timestamp for target message
- Group reactions require targetAuthor or targetAuthorUuid

Config:
- channels.signal.actions.reactions: enable/disable (default true)
- channels.signal.reactionLevel: off | ack | minimal | extensive

## Delivery targets (CLI/cron)

- DMs: signal:+15551234567 (or plain E.164)
- UUID DMs: uuid: (or bare UUID)
- Groups: signal:group:
- Usernames: username: (if supported by account)

## Troubleshooting

Run diagnostic ladder:
```bash
openclaw status
openclaw gateway status
openclaw logs --follow
openclaw doctor
openclaw channels status --probe
```

Then confirm DM pairing state:
```bash
openclaw pairing list signal
```

Common failures:
- Daemon reachable but silent: verify account/daemon settings and receive mode
- DMs ignored: sender pending pairing approval
- Group messages ignored: group sender/mention gating blocks delivery

## Configuration reference (Signal)

Full configuration: [Configuration](/gateway/configuration)
Provider options:

- channels.signal.enabled: enable/disable channel startup
- channels.signal.account: E.164 for bot account
- channels.signal.cliPath: path to signal-cli
- channels.signal.httpUrl: full daemon URL (overrides host/port)
- channels.signal.httpHost, channels.signal.httpPort: daemon bind (default 127.0.0.1:8080)
- channels.signal.autoStart: auto-spawn daemon (default true if httpUrl unset)
- channels.signal.startupTimeoutMs: startup wait timeout in ms (cap 120000)
- channels.signal.receiveMode: on-start | manual
- channels.signal.ignoreAttachments: skip attachment downloads
- channels.signal.ignoreStories: ignore stories from daemon
- channels.signal.sendReadReceipts: forward read receipts
- channels.signal.dmPolicy: pairing | allowlist | open | disabled (default: pairing)
- channels.signal.allowFrom: DM allowlist (E.164 or uuid:). open requires "*"
- channels.signal.groupPolicy: open | allowlist | disabled (default: allowlist)
- channels.signal.groupAllowFrom: group sender allowlist
- channels.signal.historyLimit: max group messages for context (0 disables)
- channels.signal.dmHistoryLimit: DM history limit in user turns
- channels.signal.textChunkLimit: outbound chunk size (chars)
- channels.signal.chunkMode: length (default) or newline
- channels.signal.mediaMaxMb: inbound/outbound media cap (MB)

Related global options:
- agents.list[].groupChat.mentionPatterns (Signal no native mentions)
- messages.groupChat.mentionPatterns (global fallback)
- messages.responsePrefix
