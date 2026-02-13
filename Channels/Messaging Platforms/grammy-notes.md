# grammY - Telegram Bot API Integration

## Why grammY

- TypeScript-first Bot API client with built-in long-poll + webhook helpers
- Middleware support and error handling
- Rate limiter and proxy support
- Cleaner media helpers than hand-rolling fetch + FormData
- Supports all Bot API methods
- Extensible and type-safe context

## What we shipped

- **Single client path:** fetch-based implementation removed; grammY is now the sole Telegram client (send + gateway) with grammY throttler enabled by default.

- **Gateway:** monitorTelegramProvider builds grammY Bot, wires mention/allowlist gating, media download via getFile/download, and delivers replies with sendMessage/sendPhoto/sendVideo/sendAudio/sendDocument. Supports long-poll or webhook via webhookCallback.

- **Proxy:** optional channels.telegram.proxy uses undici.ProxyAgent through grammY's client.baseFetch.

- **Webhook support:** webhook-set.ts wraps setWebhook/deleteWebhook; webhook.ts hosts callback with health + graceful shutdown. Gateway enables webhook mode when channels.telegram.webhookUrl + channels.telegram.webhookSecret are set (otherwise long-polls).

- **Sessions:** direct chats collapse into agent main session (agent::); groups use agent::telegram:group:; replies route back to same channel.

- **Config knobs:**
  - channels.telegram.botToken
  - channels.telegram.dmPolicy
  - channels.telegram.groups (allowlist + mention defaults)
  - channels.telegram.allowFrom
  - channels.telegram.groupAllowFrom
  - channels.telegram.groupPolicy
  - channels.telegram.mediaMaxMb
  - channels.telegram.linkPreview
  - channels.telegram.proxy
  - channels.telegram.webhookSecret
  - channels.telegram.webhookUrl

- **Draft streaming:** optional channels.telegram.streamMode uses sendMessageDraft in private topic chats (Bot API 9.3+). Separate from channel block streaming.

- **Tests:** grammY mocks cover DM + group mention gating and outbound send; media/webhook fixtures welcome.

## Open questions

- Optional grammY plugins (throttler) if hitting Bot API 429s
- Add more structured media tests (stickers, voice notes)
- Make webhook listen port configurable (currently 8787 unless wired through gateway)
