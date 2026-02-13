# OpenClaw Channels - Overview

OpenClaw can talk to you on any chat app you already use. Each channel connects via the Gateway.
Text is supported everywhere; media and reactions vary by channel.

## Messaging Platforms

### Core/Built-in
- **[WhatsApp](./Messaging%20Platforms/whatsapp.md)** — Most popular; uses Baileys and requires QR pairing.
- **[Telegram](./Messaging%20Platforms/telegram.md)** — Bot API via grammY; supports groups.
- **[Discord](./Messaging%20Platforms/discord.md)** — Discord Bot API + Gateway; supports servers, channels, and DMs.
- **[IRC](./Messaging%20Platforms/irc.md)** — Classic IRC servers; channels + DMs with pairing/allowlist controls.
- **[Slack](./Messaging%20Platforms/slack.md)** — Bolt SDK; workspace apps.
- **[Signal](./Messaging%20Platforms/signal.md)** — signal-cli; privacy-focused.
- **[WebChat](./Messaging%20Platforms/webchat.md)** — Gateway WebChat UI over WebSocket.

### Plugins (Installed Separately)
- **[Feishu](./Messaging%20Platforms/feishu.md)** — Feishu/Lark bot via WebSocket (plugin).
- **[Google Chat](./Messaging%20Platforms/google-chat.md)** — Google Chat API app via HTTP webhook.
- **[Mattermost](./Messaging%20Platforms/mattermost.md)** — Bot API + WebSocket; channels, groups, DMs (plugin).
- **[BlueBubbles](./Messaging%20Platforms/bluebubbles.md)** — Recommended for iMessage; uses BlueBubbles macOS server REST API with full feature support.
- **[iMessage (legacy)](./Messaging%20Platforms/imessage-legacy.md)** — Legacy macOS integration via imsg CLI (deprecated, use BlueBubbles).
- **[Microsoft Teams](./Messaging%20Platforms/microsoft-teams.md)** — Bot Framework; enterprise support (plugin).
- **[LINE](./Messaging%20Platforms/line.md)** — LINE Messaging API bot (plugin).
- **[Nextcloud Talk](./Messaging%20Platforms/nextcloud-talk.md)** — Self-hosted chat via Nextcloud Talk (plugin).
- **[Matrix](./Messaging%20Platforms/matrix.md)** — Matrix protocol with E2EE support (plugin).
- **[Nostr](./Messaging%20Platforms/nostr.md)** — Decentralized DMs via NIP-04 (plugin).
- **[Tlon](./Messaging%20Platforms/tlon.md)** — Urbit-based messenger (plugin).
- **[Twitch](./Messaging%20Platforms/twitch.md)** — Twitch chat via IRC connection (plugin).
- **[Zalo](./Messaging%20Platforms/zalo.md)** — Zalo Bot API; Vietnam's popular messenger (plugin).
- **[Zalo Personal](./Messaging%20Platforms/zalo-personal.md)** — Zalo personal account via QR login (plugin).

## Advanced Topics

- **[Messaging Platforms / Groups](./Messaging%20Platforms/groups.md)** — Consistent group chat handling across all surfaces
- **[Messaging Platforms / Security](./Messaging%20Platforms/security.md)** — DM access, plugin safety, prompt injection awareness
- **[Messaging Platforms / grammY Notes](./Messaging%20Platforms/grammy-notes.md)** — Telegram's grammY integration internals

## Configuration

- **[Troubleshooting](./Configuration/troubleshooting.md)** — Diagnostic ladder and channel-specific failure signatures
- **Pairing** — DM pairing + allowlists (coming soon)
- **Group Messages** — Message routing and mention gating (coming soon)
- **Groups** — Group policy and allowlist configuration (coming soon)
- **Broadcast Groups** — Broadcast chat handling (coming soon)
- **Channel Routing** — Multi-channel and multi-agent routing (coming soon)
- **Channel Location Parsing** — Extracting location data from messages (coming soon)

## Key Concepts

### Text Support
Text is supported on all channels. Media (images, files, voice) and reactions vary by channel (see individual channel docs).

### Multi-Channel
Channels can run simultaneously; configure multiple and OpenClaw will route per chat.

### Fastest Setup
Usually Telegram (simple bot token). WhatsApp requires QR pairing and stores more state on disk.

### Group Behavior
Varies by channel; see [Groups](./Messaging%20Platforms/groups.md).

### DM & Access Control
DM pairing and allowlists are enforced for safety; see [Security](./Messaging%20Platforms/security.md).

### Telegram Internals
[grammY Notes](./Messaging%20Platforms/grammy-notes.md) for implementation details.

### Troubleshooting
[Channel troubleshooting](./Configuration/troubleshooting.md) for diagnostic steps.

### Model Providers
Documented separately; see [Model Providers](/providers/models).
