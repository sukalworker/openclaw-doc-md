# Discord (Bot API)

Status: ready for DMs and guild channels via the official Discord gateway.

## Quick setup

1. Create Discord bot in Developer Portal
2. Copy bot token and intents
3. Configure OpenClaw
4. Invite bot to your guild/server

## Runtime model

- Gateway owns the Discord connection
- Reply routing deterministic: Discord inbound replies to Discord
- DMs share agent main session (agent:main:main) by default
- Guild channels isolated (agent::discord:channel:)
- Group DMs ignored by default (channels.discord.dm.groupEnabled=false)
- Slash commands run in isolated command sessions (agent::discord:slash:)

## Access control and routing

### DM policy

channels.discord.dm.policy controls DM access:

- **pairing** (default)
- **allowlist**
- **open** (requires channels.discord.dm.allowFrom to include "*")
- **disabled**

DM target format: user: or mention

### Guild handling

channels.discord.groupPolicy controls guild access:

- **open**
- **allowlist** (default, if channels.discord exists)
- **disabled**

### Guild allowlist behavior

When groupPolicy="allowlist":
- Guild must match channels.discord.guilds (id preferred, slug accepted)
- Optional sender allowlists: users (IDs or names) and roles (role IDs only)
- If either configured, senders allowed when matching users OR roles
- If guild has channels configured, non-listed channels denied
- If guild has no channels block, all channels in allowlisted guild allowed

### Mention gating

Guild messages mention-gated by default. Mention includes:
- Explicit bot mention
- Configured mention patterns (agents.list[].groupChat.mentionPatterns, fallback messages.groupChat.mentionPatterns)
- Implicit reply-to-bot behavior

requireMention configured per guild/channel (channels.discord.guilds...).

### Group DMs

Default: ignored (dm.groupEnabled=false)
Optional allowlist via dm.groupChannels (channel IDs or slugs)

### Role-based agent routing

Use bindings[].match.roles to route Discord guild members to different agents by role ID.

## Developer Portal setup

Bot registration, intents, permissions covered in full docs.

## Native commands and command auth

commands.native defaults to "auto" and enabled for Discord.
Per-channel override: channels.discord.commands.native.
Native command auth uses same Discord allowlists/policies as normal message handling.

## Feature details

Messaging, channel admin, moderation, presence, and metadata actions supported.
Media, reactions, threads, pins, polls, search, emoji/sticker management supported.

## Tools and action gates

Action gates under channels.discord.actions.*.
Default: reactions, messages, threads, pins, polls enabled; roles, moderation, presence disabled.

## Troubleshooting

Run diagnostics: openclaw channels status --probe
Common issues: guild/channel allowlist, mention gating, intents, permissions.

## Configuration reference pointers

- [Configuration reference - Discord](/gateway/configuration-reference#discord)
- startup/auth: enabled, token, accounts.*, allowBots
- policy: groupPolicy, dm.*, guilds.*, guilds.*.channels.*
- command: commands.native, commands.useAccessGroups, configWrites
- reply/history: replyToMode, historyLimit, dmHistoryLimit, dms.*.historyLimit
- delivery: textChunkLimit, chunkMode, maxLinesPerMessage
- media/retry: mediaMaxMb, retry
- actions: actions.*
- features: pluralkit, execApprovals, intents, agentComponents, heartbeat, responsePrefix
- [Pairing](/channels/pairing)
- [Channel routing](/channels/channel-routing)
- [Troubleshooting](/channels/troubleshooting)
- [Slash commands](/tools/slash-commands)
