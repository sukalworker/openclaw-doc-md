# Channel Routing

Channel routing determines how inbound messages from different channels map to agent sessions and how the model decides where replies go.

## Deterministic Routing

OpenClaw uses **deterministic routing**: replies always go back to the channel they arrived on. The model does not choose channels.

This is a key safety feature — prevents the bot from accidentally messaging the wrong person or group.

## Session Keys

Each message creates or updates a session identified by a unique session key. Session keys follow this pattern:

```
agent:<agent-id>:<session-type>:<session-id>
```

### Session Types

**Direct Messages:**
- Key: `agent:main:main` (default)
- Or: `agent:main:per-channel-peer:<channel>:<peer-id>` (isolated DM mode)
- All DMs share a single session by default (for conversational continuity)
- Can be isolated per channel+sender if configured (see below)

**Group/Channel Messages:**
- Key: `agent:main:<channel>:group:<group-id>`
- Each group gets isolated session (prevents context leakage between groups)
- Telegram forum topics add `:topic:<topic-id>` for per-topic isolation

**Control Commands:**
- Key: `agent:main:<channel>:command:<command-id>`
- Slash commands run in isolated sessions
- Still route reply back to target conversation session

## DM Session Isolation

By default, all DMs collapse into the main session:
```json
{
  "session": {
    "dmScope": "main"
  }
}
```

If multiple people can DM the bot (open DMs or large allowlist), isolate per person:
```json
{
  "session": {
    "dmScope": "per-channel-peer"
  }
}
```

This prevents one person's conversation from leaking context to another.

### Collapsing DMs Across Channels

If same person contacts you on multiple channels (Telegram, Discord, WhatsApp), collapse those into one logical identity:
```json
{
  "session": {
    "identityLinks": [
      {
        "sources": [
          { "channel": "telegram", "peerId": "123456789" },
          { "channel": "discord", "peerId": "987654321" }
        ],
        "targetIdentity": "user:john@example.com"
      }
    ]
  }
}
```

## Multi-Agent Routing (Bindings)

Route different people or channels to different agents using bindings:

```json
{
  "agents": {
    "list": [
      { "id": "main" },
      { "id": "assistant-opus", "workspace": "/path/to/opus" }
    ]
  },
  "bindings": [
    {
      "agentId": "main",
      "match": {
        "channel": "telegram",
        "peer": { "kind": "direct", "id": "123456789" }
      }
    },
    {
      "agentId": "assistant-opus",
      "match": {
        "channel": "telegram",
        "peer": { "kind": "direct", "id": "987654321" }
      }
    }
  ]
}
```

### Binding Match Fields

- **channel**: telegram, discord, slack, whatsapp, signal, imessage, etc.
- **peer**: { kind: "direct" | "group", id: "<peer-id>" }
- **guildId**: (Discord) guild/server ID
- **roles**: (Discord) array of role IDs (matches if user has any role)

## Group vs Direct Routing

Groups always map to isolated sessions (agent::<channel>:group:<id>), even if dmScope=main.

This prevents group context from leaking into your DMs and vice versa.

## Reply Routing

Replies always go back to the inbound channel and recipient:

- **Inbound from Telegram DM** → Reply goes to same Telegram DM
- **Inbound from Discord server** → Reply goes to same Discord server
- **Inbound from WhatsApp group** → Reply goes to same group

The model cannot change the destination channel.

## Threading (Channel-Specific)

Some channels support threaded replies:
- **Slack**: replyToMode (off | first | all)
- **Discord**: replyToMode (off | first | all)
- **Matrix**: threadReplies (off | inbound | always)
- **Telegram**: forum topics auto-isolated

Threading keeps replies organized but doesn't change routing — replies still go to same channel.

## History & Context

History context is per-session. When routing messages:
- **DM history** from user A doesn't leak to user B (if isolated)
- **Group history** doesn't leak to DMs
- **Channel A history** doesn't leak to channel B

Configure context depth per channel:
- channels.<name>.historyLimit (group context)
- channels.<name>.dmHistoryLimit (DM context)
- messages.groupChat.historyLimit (global default)

## Example: Routing Strategy

Typical setup:
```json
{
  "session": {
    "dmScope": "per-channel-peer"
  },
  "agents": {
    "list": [
      { "id": "main" },
      { "id": "support", "workspace": "/agents/support" }
    ]
  },
  "bindings": [
    {
      "agentId": "main",
      "match": { "channel": "telegram" }
    },
    {
      "agentId": "support",
      "match": { "channel": "discord", "guildId": "support-guild-id" }
    }
  ]
}
```

This setup:
- Routes all Telegram to "main" agent
- Routes Discord support guild to "support" agent
- Isolates DMs per person (prevents context leakage)
- Groups always get their own sessions (by default)

## Security Implications

**Deterministic routing prevents:**
- Bot accidentally messaging wrong person
- Model choosing to message public channel (even if instructed)
- Context leakage between users/groups

**Multi-agent routing allows:**
- Different policies per person/channel (tools, models, system prompts)
- Sandboxing public-facing agents separately from personal agents

See [Security](/channels/security) for threat model and config examples.
