# Slack

Status: production-ready for DMs + channels via Slack app integrations. Default mode is Socket Mode; HTTP Events API mode also supported.

## Quick setup

### Socket Mode (default)
1. Create Slack app at api.slack.com
2. Enable Socket Mode
3. Generate app-level token (xapp-...)
4. Generate bot token (xoxb-...)
5. Configure OpenClaw and start gateway

### HTTP Events API mode
1. Create Slack app
2. Enable Events API
3. Set request URL (public webhook)
4. Generate bot token
5. Configure and restart

## Token model

- botToken + appToken required for Socket Mode
- HTTP mode requires botToken + signingSecret
- Config tokens override env fallback
- SLACK_BOT_TOKEN / SLACK_APP_TOKEN env fallback applies only to default account
- userToken (xoxp-...) config-only (no env fallback), defaults to read-only (userTokenReadOnly: true)

## Access control and routing

### DM policy

channels.slack.dm.policy controls DM access:

- **pairing** (default)
- **allowlist**
- **open** (requires dm.allowFrom to include "*")
- **disabled**

DM flags:
- dm.enabled (default true)
- dm.allowFrom
- dm.groupEnabled (group DMs default false)
- dm.groupChannels (optional MPIM allowlist)

Pairing: `openclaw pairing approve slack <code>`

### Channel policy

channels.slack.groupPolicy controls channel handling:

- **open**
- **allowlist**
- **disabled**

Channel allowlist under channels.slack.channels.

Name/ID resolution at startup when token access allows; unresolved entries kept as configured.

### Mention gating

Channel messages mention-gated by default. Mention sources:
- Explicit app mention ()
- Mention regex patterns (agents.list[].groupChat.mentionPatterns, fallback messages.groupChat.mentionPatterns)
- Implicit reply-to-bot thread behavior

Per-channel controls (channels.slack.channels.<id|name>):
- requireMention
- users (allowlist)
- allowBots
- skills
- systemPrompt
- tools, toolsBySender

## Commands and slash behavior

- Native command auto-mode is off for Slack (commands.native: "auto" does not enable Slack native commands)
- Enable with channels.slack.commands.native: true
- Single slash command via channels.slack.slashCommand (defaults to "/openclaw", ephemeral: true)

Slash sessions use isolated keys (agent::slack:slash:) and route to target conversation session.
- DMs → direct; channels → channel; MPIMs → group
- Default session.dmScope=main collapses Slack DMs to agent main session
- Channel sessions: agent::slack:channel:
- Thread replies can create thread session suffixes (:thread:) when applicable

Reply threading controls:
- channels.slack.replyToMode: off|first|all (default off)
- channels.slack.replyToModeByChatType: per direct|group|channel
- Legacy: channels.slack.dm.replyToMode

Manual reply tags supported:
- [[reply_to_current]]
- [[reply_to:]]

## Actions and gates

Slack actions controlled by channels.slack.actions.*
Available action groups: messages, reactions, pins, memberInfo, emojiList (all enabled by default)

## Events and operational behavior

- Message edits/deletes/thread broadcasts mapped to system events
- Reaction add/remove events mapped to system events
- Member join/leave, channel created/renamed, pin events mapped to system events
- channel_id_changed can migrate channel config keys when configWrites enabled
- Channel topic/purpose metadata treated as untrusted context

## Manifest and scope checklist

Full manifest and OAuth scope details in extended documentation.

## Troubleshooting

Common issues: Socket Mode auth, scopes, token validity, channel allowlist.

## Configuration reference pointers

- [Configuration reference - Slack](/gateway/configuration-reference#slack)
- mode/auth: mode, botToken, appToken, signingSecret, webhookPath, accounts.*
- DM access: dm.enabled, dm.policy, dm.allowFrom, dm.groupEnabled, dm.groupChannels
- channel access: groupPolicy, channels.*, channels.*.users, channels.*.requireMention
- threading/history: replyToMode, replyToModeByChatType, thread.*, historyLimit, dmHistoryLimit, dms.*.historyLimit
- delivery: textChunkLimit, chunkMode, mediaMaxMb
- ops/features: configWrites, commands.native, slashCommand.*, actions.*, userToken, userTokenReadOnly
- [Pairing](/channels/pairing)
- [Channel routing](/channels/channel-routing)
- [Troubleshooting](/channels/troubleshooting)
- [Configuration](/gateway/configuration)
- [Slash commands](/tools/slash-commands)
