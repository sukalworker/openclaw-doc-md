# Broadcast Groups

Broadcast groups are channels where OpenClaw can send messages to multiple recipients without expecting replies.

Status: Not yet implemented in OpenClaw core. Planned for future releases.

## What They Are

A broadcast group is a target configuration that allows you to send a single message to multiple channels/users simultaneously via CLI, cron jobs, or agent actions.

Example use case: send a daily summary to multiple people across different channels (Telegram, Discord, WhatsApp).

## Planned Feature

When implemented, broadcast groups will support:
- Defining lists of recipients across multiple channels
- Broadcasting a message to all recipients in one command
- Conditional broadcasting (e.g., only on weekdays)
- Per-recipient customization (templating)

Example config (planned):
```json
{
  "broadcast": {
    "groups": {
      "daily_summary": [
        { "channel": "telegram", "target": "+1-555-123-4567" },
        { "channel": "discord", "target": "user:12345678" },
        { "channel": "whatsapp", "target": "+1-555-another-person" }
      ]
    }
  }
}
```

Usage (planned):
```bash
openclaw message broadcast --group daily_summary --message "Daily summary..."
```

## Alternatives (Today)

For now, use:
- **Cron jobs** with `message send` to each target separately
- **Slash commands** in groups (everyone sees the same message)
- **Multiple message send calls** in scripts or cron jobs

Example cron job (current workaround):
```bash
openclaw message send --channel telegram --target "+1-555-person1" --message "Summary..."
openclaw message send --channel discord --target "user:12345678" --message "Summary..."
openclaw message send --channel whatsapp --target "+1-555-person2" --message "Summary..."
```

## Future Roadmap

Broadcast groups are planned for an upcoming release. Current priority is stabilizing:
- Core channels and routing
- Security and access control
- Multi-agent support

Check GitHub issues for roadmap updates: https://github.com/openclaw/openclaw
