# Nostr (plugin)

Status: Optional plugin (disabled by default).

Nostr is decentralized protocol for social networking. This channel enables OpenClaw to receive and respond to encrypted direct messages (DMs) via NIP-04.

## Install (on demand)

### Onboarding (recommended)

Onboarding wizard (`openclaw onboard`) and `openclaw channels add` list optional channel plugins.

Selecting Nostr prompts to install plugin on demand.

Install defaults:
- Dev channel + git checkout available: uses local plugin path
- Stable/Beta: downloads from npm

You can always override the choice.

### Manual install

```bash
openclaw plugins install @openclaw/nostr
```

Use local checkout (dev workflows):
```bash
openclaw plugins install --link <path-to-openclaw>/extensions/nostr
```

Restart Gateway after installing or enabling plugins.

## Quick setup

Generate Nostr keypair (if needed):
```bash
# Using nak
nak key generate
```

Add to config:
```json
{
  "channels": {
    "nostr": {
      "privateKey": "${NOSTR_PRIVATE_KEY}"
    }
  }
}
```

Export the key:
```bash
export NOSTR_PRIVATE_KEY="nsec1..."
```

Restart Gateway.

## Configuration reference

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| privateKey | string | required | Private key in nsec or hex format |
| relays | string[] | ['wss://relay.damus.io', 'wss://nos.lol'] | Relay URLs (WebSocket) |
| dmPolicy | string | pairing | DM access policy |
| allowFrom | string[] | [] | Allowed sender pubkeys |
| enabled | boolean | true | Enable/disable channel |
| name | string | - | Display name |
| profile | object | - | NIP-01 profile metadata |

Profile data published as NIP-01 kind:0 event. Manage from Control UI (Channels -> Nostr -> Profile) or set directly in config.

Example:
```json
{
  "channels": {
    "nostr": {
      "privateKey": "${NOSTR_PRIVATE_KEY}",
      "profile": {
        "name": "openclaw",
        "displayName": "OpenClaw",
        "about": "Personal assistant DM bot",
        "picture": "https://example.com/avatar.png",
        "banner": "https://example.com/banner.png",
        "website": "https://example.com",
        "nip05": "user@example.com",
        "lud16": "user@example.com"
      }
    }
  }
}
```

Notes:
- Profile URLs must use https://
- Importing from relays merges fields and preserves local overrides

## Access control

### DM policies

- **pairing** (default): unknown senders get pairing code
- **allowlist**: only pubkeys in allowFrom can DM
- **open**: public inbound DMs (requires allowFrom: ["*"])
- **disabled**: ignore inbound DMs

### Allowlist example

```json
{
  "channels": {
    "nostr": {
      "privateKey": "${NOSTR_PRIVATE_KEY}",
      "dmPolicy": "allowlist",
      "allowFrom": ["npub1abc...", "npub1xyz..."]
    }
  }
}
```

## Key formats

Accepted formats:
- Private key: nsec... or 64-char hex
- Pubkeys (allowFrom): npub... or hex

## Relays

Defaults: relay.damus.io and nos.lol.

```json
{
  "channels": {
    "nostr": {
      "privateKey": "${NOSTR_PRIVATE_KEY}",
      "relays": ["wss://relay.damus.io", "wss://relay.primal.net", "wss://nostr.wine"]
    }
  }
}
```

Tips:
- Use 2-3 relays for redundancy
- Avoid too many relays (latency, duplication)
- Paid relays can improve reliability
- Local relays fine for testing (ws://localhost:7777)

## Protocol support

| NIP | Status | Description |
|-----|--------|-------------|
| NIP-01 | Supported | Basic event format + profile metadata |
| NIP-04 | Supported | Encrypted DMs (kind:4) |
| NIP-17 | Planned | Gift-wrapped DMs |
| NIP-44 | Planned | Versioned encryption |

## Testing

### Local relay

```bash
# Start strfry
docker run -p 7777:7777 ghcr.io/hoytech/strfry
```

Config:
```json
{
  "channels": {
    "nostr": {
      "privateKey": "${NOSTR_PRIVATE_KEY}",
      "relays": ["ws://localhost:7777"]
    }
  }
}
```

### Manual test

- Note bot pubkey (npub) from logs
- Open Nostr client (Damus, Amethyst, etc.)
- DM bot pubkey
- Verify response

## Troubleshooting

### Not receiving messages

- Verify private key is valid
- Ensure relay URLs reachable and use wss:// (or ws:// for local)
- Confirm enabled is not false
- Check Gateway logs for relay connection errors

### Not sending responses

- Check relay accepts writes
- Verify outbound connectivity
- Watch for relay rate limits

### Duplicate responses

- Expected when using multiple relays
- Messages deduplicated by event ID; only first delivery triggers response

## Security

- Never commit private keys
- Use environment variables for keys
- Consider allowlist for production bots

## Limitations (MVP)

- Direct messages only (no group chats)
- No media attachments
- NIP-04 only (NIP-17 gift-wrap planned)
