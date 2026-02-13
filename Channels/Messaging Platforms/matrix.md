# Matrix (plugin)

Matrix is open, decentralized messaging protocol. OpenClaw connects as Matrix user on any homeserver, so you need Matrix account for bot. Once logged in, DM bot directly or invite it to rooms (Matrix "groups"). Beeper valid client option, but requires E2EE enabled.

Status: supported via plugin (@vector-im/matrix-bot-sdk). Direct messages, rooms, threads, media, reactions, polls (send + poll-start as text), location, and E2EE (with crypto support) supported.

## Plugin required

Matrix ships as plugin, not bundled with core.

Install via CLI (npm registry):
```bash
openclaw plugins install @openclaw/matrix
```

Local checkout (git repo):
```bash
openclaw plugins install ./extensions/matrix
```

If you choose Matrix during configure/onboarding and git checkout detected, OpenClaw will offer local install path automatically.

Details: [Plugins](/tools/plugin)

## Setup

- Install Matrix plugin: `openclaw plugins install @openclaw/matrix` or local checkout
- Create Matrix account on homeserver: Browse hosting at [https://matrix.org/ecosystem/hosting/](https://matrix.org/ecosystem/hosting/) or host yourself
- Get access token for bot account:

Use Matrix login API with curl at your homeserver:
```bash
curl --request POST \
  --url https://matrix.example.org/_matrix/client/v3/login \
  --header 'Content-Type: application/json' \
  --data '{
    "type": "m.login.password",
    "identifier": {
      "type": "m.id.user",
      "user": "your-user-name"
    },
    "password": "your-password"
  }'
```

Replace matrix.example.org with your homeserver URL.

Or set channels.matrix.userId + channels.matrix.password: OpenClaw calls same login endpoint, stores access token in ~/.openclaw/credentials/matrix/credentials.json, and reuses it on next start.

- Configure credentials:
  - Env: MATRIX_HOMESERVER, MATRIX_ACCESS_TOKEN (or MATRIX_USER_ID + MATRIX_PASSWORD)
  - Or config: channels.matrix.*
- If both set, config takes precedence
- With access token: user ID fetched automatically via /whoami
- When set, channels.matrix.userId should be full Matrix ID (example: @bot:example.org)
- Restart gateway (or finish onboarding)
- Start DM with bot or invite it to room from any Matrix client (Element, Beeper, etc.; see [https://matrix.org/ecosystem/clients/](https://matrix.org/ecosystem/clients/))

Beeper requires E2EE, so set channels.matrix.encryption: true and verify device.

Minimal config (access token, user ID auto-fetched):
```json
{
  "channels": {
    "matrix": {
      "enabled": true,
      "homeserver": "https://matrix.example.org",
      "accessToken": "syt_***",
      "dm": { "policy": "pairing" }
    }
  }
}
```

E2EE config:
```json
{
  "channels": {
    "matrix": {
      "enabled": true,
      "homeserver": "https://matrix.example.org",
      "accessToken": "syt_***",
      "encryption": true,
      "dm": { "policy": "pairing" }
    }
  }
}
```

## Encryption (E2EE)

End-to-end encryption supported via Rust crypto SDK.

Enable with channels.matrix.encryption: true:
- If crypto module loads, encrypted rooms decrypted automatically
- Outbound media encrypted when sending to encrypted rooms
- On first connection, OpenClaw requests device verification from other sessions
- Verify device in another Matrix client (Element, etc.) to enable key sharing
- If crypto module cannot load, E2EE disabled and encrypted rooms won't decrypt; OpenClaw logs warning

If missing crypto module errors (e.g., @matrix-org/matrix-sdk-crypto-nodejs-*):
- Allow build scripts for @matrix-org/matrix-sdk-crypto-nodejs
- Run `pnpm rebuild @matrix-org/matrix-sdk-crypto-nodejs`
- Or fetch binary with `node node_modules/@matrix-org/matrix-sdk-crypto-nodejs/download-lib.js`

Crypto state stored per account + access token in ~/.openclaw/matrix/accounts/<account>/<homeserver>__<user>/<token-hash>/crypto/ (SQLite database). Sync state lives alongside in bot-storage.json.

If access token (device) changes, new store created and bot must be re-verified for encrypted rooms.

**Device verification:**
When E2EE enabled, bot requests verification from other sessions on startup.
Open Element (or another client) and approve verification request to establish trust.
Once verified, bot can decrypt messages in encrypted rooms.

## Routing model

- Replies always go back to Matrix
- DMs share agent main session; rooms map to group sessions

## Access control (DMs)

- Default: channels.matrix.dm.policy = "pairing" (unknown senders get pairing code)
- Approve: `openclaw pairing list matrix` / `openclaw pairing approve matrix`
- Public DMs: channels.matrix.dm.policy="open" plus channels.matrix.dm.allowFrom=["*"]
- channels.matrix.dm.allowFrom accepts full Matrix user IDs (example: @user:server). Wizard resolves display names when directory search finds single exact match.

## Rooms (groups)

- Default: channels.matrix.groupPolicy = "allowlist" (mention-gated). Use channels.defaults.groupPolicy to override when unset.
- Allowlist rooms with channels.matrix.groups (room IDs or aliases; names resolved to IDs when directory search finds single exact match):

```json
{
  "channels": {
    "matrix": {
      "groupPolicy": "allowlist",
      "groups": {
        "!roomId:example.org": { "allow": true },
        "#alias:example.org": { "allow": true }
      },
      "groupAllowFrom": ["@owner:example.org"]
    }
  }
}
```

- requireMention: false enables auto-reply in that room
- groups."*" can set defaults for mention gating
- groupAllowFrom restricts which senders can trigger (full Matrix user IDs)
- Per-room users allowlists further restrict senders inside specific room (use full Matrix user IDs)
- Configure wizard prompts for room allowlists (room IDs, aliases, or names); resolves names only on exact unique match
- On startup, OpenClaw resolves room/user names in allowlists to IDs and logs mapping; unresolved entries ignored
- Invites auto-joined by default; control with channels.matrix.autoJoin and channels.matrix.autoJoinAllowlist
- To allow no rooms, set channels.matrix.groupPolicy: "disabled" (or keep empty allowlist)
- Legacy key: channels.matrix.rooms (same shape as groups)

## Threads

- Reply threading supported
- channels.matrix.threadReplies controls whether replies stay in threads: off, inbound (default), always
- channels.matrix.replyToMode controls reply-to metadata when not replying in thread: off (default), first, all

## Capabilities

| Feature | Status |
|---------|--------|
| Direct messages | ✅ Supported |
| Rooms | ✅ Supported |
| Threads | ✅ Supported |
| Media | ✅ Supported |
| E2EE | ✅ Supported (crypto module required) |
| Reactions | ✅ Supported (send/read via tools) |
| Polls | ✅ Send supported; inbound poll starts converted to text |
| Location | ✅ Supported (geo URI; altitude ignored) |
| Native commands | ✅ Supported |

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
openclaw pairing list matrix
```

Common failures:
- Logged in but room messages ignored: room blocked by groupPolicy or room allowlist
- DMs ignored: sender pending approval when channels.matrix.dm.policy="pairing"
- Encrypted rooms fail: crypto support or encryption settings mismatch

For triage flow: [/channels/troubleshooting](/channels/troubleshooting)

## Configuration reference (Matrix)

Full configuration: [Configuration](/gateway/configuration)

Provider options:
- channels.matrix.enabled: enable/disable channel startup
- channels.matrix.homeserver: homeserver URL
- channels.matrix.userId: Matrix user ID (optional with access token)
- channels.matrix.accessToken: access token
- channels.matrix.password: password for login (token stored)
- channels.matrix.deviceName: device display name
- channels.matrix.encryption: enable E2EE (default: false)
- channels.matrix.initialSyncLimit: initial sync limit
- channels.matrix.threadReplies: off | inbound | always (default: inbound)
- channels.matrix.textChunkLimit: outbound text chunk size (chars)
- channels.matrix.chunkMode: length (default) or newline
- channels.matrix.dm.policy: pairing | allowlist | open | disabled (default: pairing)
- channels.matrix.dm.allowFrom: DM allowlist (full Matrix user IDs). open requires "*"
- channels.matrix.groupPolicy: allowlist | open | disabled (default: allowlist)
- channels.matrix.groupAllowFrom: allowlisted senders for group messages (full Matrix user IDs)
- channels.matrix.allowlistOnly: force allowlist rules for DMs + rooms
- channels.matrix.groups: group allowlist + per-room settings map
- channels.matrix.rooms: legacy group allowlist/config
- channels.matrix.replyToMode: reply-to mode for threads/tags
- channels.matrix.mediaMaxMb: inbound/outbound media cap (MB)
- channels.matrix.autoJoin: invite handling (always | allowlist | off, default: always)
- channels.matrix.autoJoinAllowlist: allowed room IDs/aliases for auto-join
- channels.matrix.actions: per-action tool gating (reactions/messages/pins/memberInfo/channelInfo)
