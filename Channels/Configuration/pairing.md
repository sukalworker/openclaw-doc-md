# Pairing

Pairing is the default DM access control mechanism across OpenClaw channels. When a channel's dmPolicy is set to "pairing", unknown senders receive a short pairing code and their messages are ignored until approved.

## How Pairing Works

1. **Unknown sender messages the bot**
2. **Bot responds with a pairing code** (e.g., `ABC123`) and instructions
3. **Sender shares the code** (e.g., via voice call, in person, etc.)
4. **You approve the pairing**: `openclaw pairing approve <channel> <code>`
5. **Sender can now message freely**

Pairing codes expire after 1 hour. Repeated messages won't resend a code until a new request is created.

## Managing Pairings

### List pending pairing requests

```bash
openclaw pairing list <channel>
```

Output shows:
- Pairing codes waiting for approval
- Sender identifiers (phone number, username, user ID, etc.)
- Code expiry time
- Any metadata available

### Approve a pairing

```bash
openclaw pairing approve <channel> <code>
```

Example:
```bash
openclaw pairing approve telegram ABC123
```

Once approved, the sender is added to the channel's allowlist and can message freely.

### View approved allowlist

Approved pairings are merged with configured allowlists and stored in:
- `~/.openclaw/credentials/<channel>-allowFrom.json`

You can also set static allowlists via config without requiring pairing (see individual channel docs).

## Channels Supporting Pairing

All major DM-capable channels support pairing:
- **WhatsApp** (dmPolicy: pairing)
- **Telegram** (dmPolicy: pairing)
- **Discord** (dm.policy: pairing)
- **Slack** (dm.policy: pairing)
- **Signal** (dmPolicy: pairing)
- **iMessage** (dmPolicy: pairing)
- **BlueBubbles** (dmPolicy: pairing)
- **Feishu** (dmPolicy: pairing)
- **Google Chat** (dm.policy: pairing)
- **Matrix** (dm.policy: pairing)
- **Nextcloud Talk** (dmPolicy: pairing)
- **Zalo** (dmPolicy: pairing)
- **Zalo Personal** (dmPolicy: pairing)
- **Nostr** (dmPolicy: pairing)

## DM Policy Options

All channels support four DM access policies:

| Policy | Behavior | When to Use |
|--------|----------|------------|
| **pairing** (default) | Unknown senders get code; ignored until approved | Default for security; good for preventing spam |
| **allowlist** | Only whitelisted senders can message | When you have a fixed list of trusted people |
| **open** | Anyone can DM; requires "*" in allowFrom | Public-facing bot; high spam/prompt injection risk |
| **disabled** | Ignore all DMs | When you only use groups/channels |

Example config:
```json
{
  "channels": {
    "telegram": {
      "dmPolicy": "pairing",
      "allowFrom": ["+1-555-123-4567"]
    }
  }
}
```

## Combining Pairing + Allowlist

You can have both:
- **Configured allowlist** (config): trusted people who don't need pairing
- **Pairing codes**: one-time approval for new people

Approved pairings are merged with config allowlist in runtime.

Example:
```json
{
  "channels": {
    "whatsapp": {
      "dmPolicy": "pairing",
      "allowFrom": ["+1-555-known-person"]
      // Unknown senders still get pairing code
    }
  }
}
```

## Security Notes

- Pairing codes are transmitted via the bot's reply (insecure channel!)
  - Best practice: have sender confirm code verbally or via trusted channel
  - Alternatively: use `allowlist` mode with pre-shared allowlists
- Approved pairings stored in `~/.openclaw/credentials/` (ensure tight file permissions: 600)
- Pairing requests are capped at 3 pending per channel (prevents spam)

## File Locations

- Pending pairings: stored in memory during gateway runtime
- Approved pairings: `~/.openclaw/credentials/<channel>-allowFrom.json`

Example content:
```json
{
  "whatsapp": ["+1-555-123-4567", "+1-555-another-person"],
  "telegram": ["123456789", "987654321"]
}
```

## Troubleshooting

**"Bot responded with pairing code but I didn't get the message"**
- Check gateway logs: `openclaw logs --follow`
- Verify the channel is sending messages (troubleshoot per channel)
- Resend a message to get a new code

**"Code expired before I could approve it"**
- Codes expire after 1 hour
- Just send another message to the bot to get a fresh code

**"I approved the pairing but still can't message"**
- Check the channel's status: `openclaw channels status`
- Verify the channel is connected and running
- Check gateway logs for approval confirmation
- May need to restart gateway for changes to take effect

**"How do I revoke a pairing?"**
- Edit `~/.openclaw/credentials/<channel>-allowFrom.json` and remove the sender's ID
- Restart gateway to reload allowlists
