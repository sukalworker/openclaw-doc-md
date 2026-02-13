# WhatsApp (Web channel)

Status: production-ready via WhatsApp Web (Baileys). Gateway owns linked session(s).

## Quick setup

1. Enable WhatsApp in config
2. Scan QR code when prompted
3. Configure DM policy and allowlists
4. Start gateway

## Deployment patterns

Multiple accounts supported via channels.whatsapp.accounts.

## Runtime model

- Gateway owns the WhatsApp socket and reconnect loop.
- Outbound sends require an active WhatsApp listener for the target account.
- Status and broadcast chats are ignored (@status, @broadcast).
- Direct chats use DM session rules (session.dmScope; default main collapses DMs to the agent main session).
- Group sessions are isolated (agent::whatsapp:group:).

## Access control and activation

### DM policy

channels.whatsapp.dmPolicy controls direct chat access:

- **pairing** (default) - Unknown senders get a pairing code
- **allowlist** - Only allowed senders
- **open** (requires allowFrom to include "*")
- **disabled** - No DMs

allowFrom accepts E.164-style numbers (normalized internally).

### Group access

Two layers:
- Group membership allowlist (channels.whatsapp.groups) - if omitted, all groups eligible; if present, acts as allowlist
- Group sender policy (channels.whatsapp.groupPolicy + groupAllowFrom) - open, allowlist, or disabled

### Mention and activation

Group replies require mention by default. Session-level activation:
- `/activation mention`
- `/activation always`

## Personal-number and self-chat behavior

When linked self number is in allowFrom, special safeguards activate:
- Skip read receipts for self-chat
- Ignore mention auto-trigger
- Self-chat replies default to response prefix

## Message normalization and context

Media downloads supported; size capped by mediaMaxMb (default 20).

## Acknowledgment reactions

Configure ackReaction for immediate receipt confirmation via emoji.

## Multi-account and credentials

Multiple accounts supported under channels.whatsapp.accounts.

## Tools, actions, and config writes

Agent tool support includes WhatsApp reaction action (react).
Action gates: channels.whatsapp.actions.reactions, channels.whatsapp.actions.polls
Channel-initiated config writes enabled by default (disable via channels.whatsapp.configWrites=false).

## Configuration reference

- [Configuration reference - WhatsApp](/gateway/configuration-reference#whatsapp)
- [Pairing](/channels/pairing)
- [Channel routing](/channels/channel-routing)
- [Troubleshooting](/channels/troubleshooting)
