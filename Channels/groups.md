# Groups

OpenClaw treats group chats consistently across surfaces: WhatsApp, Telegram, Discord, Slack, Signal, iMessage, Microsoft Teams.

## Beginner intro (2 minutes)

OpenClaw "lives" on your own messaging accounts. No separate bot user.
If you're in a group, OpenClaw can see that group and respond there.

Default behavior:
- Groups are restricted (groupPolicy: "allowlist")
- Replies require mention unless explicitly disabled

Translation: allowlisted senders can trigger OpenClaw by mentioning it.

### TL;DR

- DM access controlled by *.allowFrom
- Group access controlled by *.groupPolicy + allowlists (*.groups, *.groupAllowFrom)
- Reply triggering controlled by mention gating (requireMention, /activation)

### Quick mental model (evaluation order for group messages)

1. groupPolicy (open/disabled/allowlist)
2. group allowlists (*.groups, *.groupAllowFrom, channel-specific allowlist)
3. mention gating (requireMention, /activation)

## If you want...

| Goal | What to set |
|------|------------|
| Allow all groups but only reply on @mentions | groups: { "*": { requireMention: true } } |
| Disable all group replies | groupPolicy: "disabled" |
| Only specific groups | groups: { "<group-id>": { ... } } (no "*" key) |
| Only you can trigger in groups | groupPolicy: "allowlist", groupAllowFrom: ["+1555..."] |

## Session keys

- Group sessions use agent:::group: keys (rooms/channels use agent:::channel:)
- Telegram forum topics add :topic: to group id for topic isolation
- Direct chats use main session (or per-sender if configured)
- Heartbeats skipped for group sessions

## Pattern: personal DMs + public groups (single agent)

Yes — works well if "personal" traffic is DMs and "public" traffic is groups.

Why: in single-agent mode, DMs typically land in main session (agent:main:main), while groups use non-main session keys (agent:main:<channel>:group:<id>). With sandboxing mode: "non-main", group sessions run in Docker while main DM session stays on-host.

Gives one agent "brain" (shared workspace + memory) but two execution postures:
- DMs: full tools (host)
- Groups: sandbox + restricted tools (Docker)

Example (DMs on host, groups sandboxed + messaging-only):
```json
{
  "agents": {
    "defaults": {
      "sandbox": {
        "mode": "non-main",
        "scope": "session",
        "workspaceAccess": "none"
      }
    }
  },
  "tools": {
    "sandbox": {
      "tools": {
        "allow": ["group:messaging", "group:sessions"],
        "deny": ["group:runtime", "group:fs", "group:ui", "nodes", "cron", "gateway"]
      }
    }
  }
}
```

## Display labels

- UI labels use displayName when available, formatted as :
- #room reserved for rooms/channels; group chats use g- (lowercase, spaces -> -, keep #@+._-)

## Group policy

Control how group/room messages handled per channel:

```json
{
  "channels": {
    "whatsapp": {
      "groupPolicy": "disabled",
      "groupAllowFrom": ["+15551234567"]
    },
    "telegram": {
      "groupPolicy": "disabled",
      "groupAllowFrom": ["123456789", "@username"]
    },
    "signal": {
      "groupPolicy": "disabled",
      "groupAllowFrom": ["+15551234567"]
    },
    "imessage": {
      "groupPolicy": "disabled",
      "groupAllowFrom": ["chat_id:123"]
    },
    "msteams": {
      "groupPolicy": "disabled",
      "groupAllowFrom": ["user@example.com"]
    },
    "discord": {
      "groupPolicy": "allowlist",
      "guilds": {
        "GUILD_ID": { "channels": { "help": { "allow": true } } }
      }
    },
    "slack": {
      "groupPolicy": "allowlist",
      "channels": { "#general": { "allow": true } }
    },
    "matrix": {
      "groupPolicy": "allowlist",
      "groupAllowFrom": ["@owner:example.org"],
      "groups": {
        "!roomId:example.org": { "allow": true },
        "#alias:example.org": { "allow": true }
      }
    }
  }
}
```

| Policy | Behavior |
|--------|----------|
| "open" | Groups bypass allowlists; mention-gating still applies |
| "disabled" | Block all group messages entirely |
| "allowlist" | Only allow groups/rooms matching configured allowlist |

## Mention gating (default)

Group messages require mention unless overridden per group. Defaults live per subsystem under *.groups."*".

```json
{
  "channels": {
    "whatsapp": {
      "groups": {
        "*": { "requireMention": true },
        "[[email protected]](/cdn-cgi/l/email-protection)": { "requireMention": false }
      }
    },
    "telegram": {
      "groups": {
        "*": { "requireMention": true },
        "123456789": { "requireMention": false }
      }
    },
    "imessage": {
      "groups": {
        "*": { "requireMention": true },
        "123": { "requireMention": false }
      }
    }
  },
  "agents": {
    "list": [
      {
        "id": "main",
        "groupChat": {
          "mentionPatterns": ["@openclaw", "openclaw", "\\+15555550123"],
          "historyLimit": 50
        }
      }
    ]
  }
}
```

Notes:
- mentionPatterns are case-insensitive regexes
- Surfaces providing explicit mentions still pass; patterns are fallback
- Per-agent override: agents.list[].groupChat.mentionPatterns
- Mention gating only enforced when mention detection possible
- Discord defaults in channels.discord.guilds."*" (per-guild overridable)
- Group history wrapped uniformly; use messages.groupChat.historyLimit for global default

## Group/channel tool restrictions (optional)

Some channels support restricting tools inside specific group/room/channel:

- tools: allow/deny tools for whole group
- toolsBySender: per-sender overrides (use "*" as wildcard)

Resolution order (most specific wins):
1. group/channel toolsBySender match
2. group/channel tools
3. default ("*") toolsBySender match
4. default ("*") tools

Example (Telegram):
```json
{
  "channels": {
    "telegram": {
      "groups": {
        "*": { "tools": { "deny": ["exec"] } },
        "-1001234567890": {
          "tools": { "deny": ["exec", "read", "write"] },
          "toolsBySender": {
            "123456789": { "alsoAllow": ["exec"] }
          }
        }
      }
    }
  }
}
```

## Group allowlists

When channels.whatsapp.groups, channels.telegram.groups, or channels.imessage.groups configured, keys act as group allowlist. Use "*" to allow all groups while setting default mention behavior.

Common intents:
- Disable all: channels.whatsapp.groupPolicy: "disabled"
- Allow specific groups (WhatsApp)
- Allow all but require mention: groups: { "*": { requireMention: true } }
- Only owner can trigger: groupPolicy: "allowlist", groupAllowFrom: ["+15551234567"]

## Activation (owner-only)

Group owners toggle per-group activation:
- `/activation mention`
- `/activation always`

Owner determined by channels.whatsapp.allowFrom (or bot self E.164 when unset).

## Context fields

Group inbound payloads set:
- ChatType=group
- GroupSubject (if known)
- GroupMembers (if known)
- WasMentioned (mention gating result)
- Telegram forum topics include MessageThreadId and IsForum

Agent system prompt includes group intro on first turn of new group session (reminds model to respond like human, avoid Markdown tables, avoid literal \\n).

## iMessage specifics

- Prefer chat_id: when routing/allowlisting
- List chats: imsg chats --limit 20
- Group replies always go back to same chat_id

## WhatsApp specifics

See [Group messages](/channels/group-messages) for WhatsApp-only behavior (history injection, mention handling details).
