# iMessage (legacy: imsg)

Status: legacy external CLI integration. Gateway spawns imsg rpc and communicates over JSON-RPC on stdio (no separate daemon/port).

**Note:** BlueBubbles is recommended for new setups. This is the legacy iMessage integration.

## Quick setup

### Local Mac (fast path)
1. Install imsg on the Mac
2. Configure OpenClaw
3. Restart gateway

### Remote Mac over SSH
Use SSH wrapper to run imsg on remote Mac. OpenClaw only requires stdio-compatible cliPath.

Example wrapper script:
```bash
#!/usr/bin/env bash
exec ssh -T gateway-host imsg "$@"
```

Recommended config when attachments enabled:
```json
{
  "channels": {
    "imessage": {
      "enabled": true,
      "cliPath": "~/.openclaw/scripts/imsg-ssh",
      "remoteHost": "user@gateway-host",
      "includeAttachments": true
    }
  }
}
```

If remoteHost not set, OpenClaw attempts auto-detect by parsing SSH wrapper script.

## Requirements and permissions (macOS)

- Messages must be signed in on Mac running imsg
- Full Disk Access required for process context running OpenClaw/imsg (Messages DB access)
- Automation permission required to send messages through Messages.app

## Access control and routing

### DM policy

channels.imessage.dmPolicy controls direct messages:
- **pairing** (default)
- **allowlist**
- **open** (requires allowFrom to include "*")
- **disabled**

Allowlist field: channels.imessage.allowFrom (handles or chat targets: chat_id:*, chat_guid:*, chat_identifier:*).

### Group policy

channels.imessage.groupPolicy controls group handling:
- **allowlist** (default when configured)
- **open**
- **disabled**

Group sender allowlist: channels.imessage.groupAllowFrom (fallback to allowFrom if unset).

### Mention gating for groups

iMessage has no native mention metadata. Mention detection uses regex patterns:
- agents.list[].groupChat.mentionPatterns
- fallback messages.groupChat.mentionPatterns

With no configured patterns, mention gating cannot be enforced.

Control commands from authorized senders can bypass mention gating in groups.

### Sessions and routing

- DMs use direct routing
- Groups use group routing
- Default session.dmScope=main collapses iMessage DMs into agent main session
- Group sessions isolated (agent::imessage:group:)
- Replies route back to iMessage using originating channel/target metadata

### Group-ish thread behavior

Some multi-participant iMessage threads arrive with is_group=false. If that chat_id explicitly configured under channels.imessage.groups, OpenClaw treats as group traffic (group gating + session isolation).

## Deployment patterns

Deployment patterns detailed in full documentation.

## Config writes

iMessage allows channel-initiated config writes by default (for /config set|unset when commands.config: true).

Disable with:
```json
{
  "channels": {
    "imessage": {
      "configWrites": false
    }
  }
}
```

## Troubleshooting

Diagnostic checklist and common issues detailed in [/channels/imessage#troubleshooting](/channels/imessage#troubleshooting).

## Configuration reference pointers

- [Configuration reference - iMessage](/gateway/configuration-reference#imessage)
- [Gateway configuration](/gateway/configuration)
- [Pairing](/channels/pairing)
- [BlueBubbles](/channels/bluebubbles) (recommended for new setups)
