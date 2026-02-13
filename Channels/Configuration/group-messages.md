# Group Messages

Group message routing and handling varies by channel. This page covers WhatsApp-specific group message behavior; for general group concepts, see [Groups](../Messaging%20Platforms/groups.md).

## WhatsApp Group Specifics

### Message Normalization

WhatsApp group messages arrive with:
- Sender phone number (E.164 format)
- Group JID (unique group identifier)
- Message text and media
- Quote/reply information (when replying to previous message)

OpenClaw normalizes these into the standard channel envelope for routing.

### History Injection

For group context, OpenClaw can inject recent message history to help the model understand conversation flow:
- channels.whatsapp.historyLimit: number of recent messages (default 50, set 0 to disable)
- History is pending messages (messages that didn't trigger reply, e.g., those missing mentions)

Example: if group has 10 messages, only 3 mentioned the bot, the remaining 7 are context history.

### Group Allowlists

WhatsApp group access controlled by two mechanisms:
1. **Group allowlist** (channels.whatsapp.groups): which groups are allowed
2. **Sender allowlist** (channels.whatsapp.groupAllowFrom): which senders can trigger bot

Combined effect: message blocked unless group allowed AND sender allowed.

### Mention Gating

By default, WhatsApp group messages require @mention to trigger reply. Non-mention messages are stored as context only.

To disable mention requirement for a specific group:
```json
{
  "channels": {
    "whatsapp": {
      "groups": {
        "123456789-987654321@g.us": {
          "requireMention": false
        }
      }
    }
  }
}
```

### Self-Chat Safeguards

When the bot's self number is configured in allowFrom (your own WhatsApp number), special safeguards activate:
- Read receipts skipped for self-chat (avoid endless loops)
- Mention auto-trigger disabled for self-messages
- Response prefix defaults to [openclaw] or configured name

## Group Message Flow

1. **Message arrives in group**
2. **Check group allowed** (groupPolicy + groups allowlist)
3. **Check sender allowed** (groupAllowFrom)
4. **Check mention requirement** (requireMention)
5. **Load history context** (historyLimit recent messages)
6. **Create group session** (agent::<channel>:group:<id>)
7. **Route to agent**
8. **Reply goes back to group** (deterministic routing)

## Common Patterns

### Allow all groups, require mention
```json
{
  "channels": {
    "whatsapp": {
      "groupPolicy": "allowlist",
      "groups": {
        "*": { "requireMention": true }
      }
    }
  }
}
```

### Allow specific groups only
```json
{
  "channels": {
    "whatsapp": {
      "groupPolicy": "allowlist",
      "groups": {
        "group1-jid@g.us": { "requireMention": true },
        "group2-jid@g.us": { "requireMention": false }
      }
    }
  }
}
```

### Only owner can trigger
```json
{
  "channels": {
    "whatsapp": {
      "groupPolicy": "allowlist",
      "groupAllowFrom": ["+1-555-owner-number"],
      "groups": {
        "*": { "requireMention": true }
      }
    }
  }
}
```

## Tools & Actions in Groups

Group tool restrictions can be set per group:
```json
{
  "channels": {
    "whatsapp": {
      "groups": {
        "123456789-987654321@g.us": {
          "tools": { "deny": ["exec", "read", "write"] }
        }
      }
    }
  }
}
```

Or per sender within a group:
```json
{
  "channels": {
    "whatsapp": {
      "groups": {
        "123456789-987654321@g.us": {
          "toolsBySender": {
            "+1-555-trusted": { "deny": [] },
            "*": { "deny": ["exec", "read", "write"] }
          }
        }
      }
    }
  }
}
```

See [Groups](../Messaging%20Platforms/groups.md) for detailed group configuration.
