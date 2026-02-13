# Telegram (Bot API)

Status: production-ready for bot DMs + groups via grammY. Long polling is the default mode; webhook mode is optional.

## Quick setup

1. Create bot with BotFather (@BotFather on Telegram)
2. Copy bot token
3. Configure OpenClaw with token
4. Start gateway

## Telegram side settings

Bot privacy mode, group privacy, and permissions can be configured via BotFather.

## Access control and activation

### DM policy

channels.telegram.dmPolicy controls direct message access:

- **pairing** (default)
- **allowlist**
- **open** (requires allowFrom to include "*")
- **disabled**

channels.telegram.allowFrom accepts numeric IDs and usernames. telegram: / tg: prefixes accepted and normalized.

### Finding your Telegram user ID

Safest: DM your bot, run `openclaw logs --follow`, read from.id
Official method: `curl "https://api.telegram.org/bot<bot_token>/getUpdates"`
Third-party: @userinfobot or @getidsbot

### Group access

Two independent controls:
- Which groups allowed (channels.telegram.groups) - no config = all allowed; if present, acts as allowlist
- Which senders allowed (channels.telegram.groupPolicy) - open, allowlist (default), or disabled

### Mention gating

Group replies require mention by default. Mention can come from:
- Native @botusername mention
- Mention patterns in agents.list[].groupChat.mentionPatterns or messages.groupChat.mentionPatterns

Session-level toggles:
- `/activation always`
- `/activation mention`

## Runtime behavior

- Telegram owned by gateway process
- Routing deterministic: Telegram inbound replies to Telegram
- Group sessions isolated by group ID (Forum topics append :topic: for topic isolation)
- DM messages preserve message_thread_id; thread-aware routing
- Long polling with per-chat/per-thread sequencing via grammY runner
- Concurrency via agents.defaults.maxConcurrent

## Feature reference

Text, media (images, files), stickers, reactions supported.
Read receipts not supported (Bot API limitation).

## Troubleshooting

Common issues and diagnostics covered in [Channel troubleshooting](/channels/troubleshooting).

## Telegram config reference pointers

- [Configuration reference - Telegram](/gateway/configuration-reference#telegram)
- startup/auth: enabled, botToken, tokenFile, accounts.*
- access control: dmPolicy, allowFrom, groupPolicy, groupAllowFrom, groups, groups.*.topics.*
- command/menu: commands.native, customCommands
- threading/replies: replyToMode
- streaming: streamMode, draftChunk, blockStreaming
- formatting/delivery: textChunkLimit, chunkMode, linkPreview, responsePrefix
- media/network: mediaMaxMb, timeoutSeconds, retry, network.autoSelectFamily, proxy
- webhook: webhookUrl, webhookSecret, webhookPath
- actions/capabilities: capabilities.inlineButtons, actions.sendMessage|editMessage|deleteMessage|reactions|sticker
- reactions: reactionNotifications, reactionLevel
- writes/history: configWrites, historyLimit, dmHistoryLimit, dms.*.historyLimit
- [Pairing](/channels/pairing)
- [Channel routing](/channels/channel-routing)
- [Troubleshooting](/channels/troubleshooting)
