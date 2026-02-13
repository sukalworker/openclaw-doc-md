# Twitch (plugin)

Twitch chat support via IRC connection. OpenClaw connects as Twitch user (bot account) to receive and send messages in channels.

## Plugin required

Twitch ships as plugin, not bundled with core.

Install via CLI (npm registry):
```bash
openclaw plugins install @openclaw/twitch
```

Local checkout (git repo):
```bash
openclaw plugins install ./extensions/twitch
```

Details: [Plugins](/tools/plugin)

## Quick setup (beginner)

- Create dedicated Twitch account for bot (or use existing)
- Generate credentials: [Twitch Token Generator](https://twitchtokengenerator.com/)
  - Select Bot Token
  - Verify scopes chat:read and chat:write selected
  - Copy Client ID and Access Token
- Find your Twitch user ID: [https://www.streamweasels.com/tools/convert-twitch-username-to-user-id/](https://www.streamweasels.com/tools/convert-twitch-username-to-user-id/)
- Configure token:
  - Env: OPENCLAW_TWITCH_ACCESS_TOKEN=... (default account only)
  - Or config: channels.twitch.accessToken
- Start gateway

⚠️ **Important:** Add access control (allowFrom or allowedRoles) to prevent unauthorized users from triggering bot. requireMention defaults to true.

Minimal config:
```json
{
  "channels": {
    "twitch": {
      "enabled": true,
      "username": "openclaw",
      "accessToken": "oauth:abc123...",
      "clientId": "xyz789...",
      "channel": "vevisk",
      "allowFrom": ["123456789"]
    }
  }
}
```

## What it is

- Twitch channel owned by Gateway
- Deterministic routing: replies always go back to Twitch
- Each account maps to isolated session key agent::twitch:
- username is bot's account (who authenticates), channel is which chat room to join

## Setup (detailed)

### Generate credentials

Use [Twitch Token Generator](https://twitchtokengenerator.com/):
- Select Bot Token
- Verify scopes chat:read and chat:write selected
- Copy Client ID and Access Token

No manual app registration needed. Tokens expire after several hours.

### Configure the bot

Env var (default account only):
```bash
OPENCLAW_TWITCH_ACCESS_TOKEN=oauth:abc123...
```

Or config:
```json
{
  "channels": {
    "twitch": {
      "enabled": true,
      "username": "openclaw",
      "accessToken": "oauth:abc123...",
      "clientId": "xyz789...",
      "channel": "vevisk"
    }
  }
}
```

If both env and config set, config takes precedence.

### Access control (recommended)

```json
{
  "channels": {
    "twitch": {
      "allowFrom": ["123456789"]
    }
  }
}
```

Prefer allowFrom for hard allowlist. Use allowedRoles instead for role-based access.

Available roles: "moderator", "owner", "vip", "subscriber", "all".

**Why user IDs?** Usernames can change, allowing impersonation. User IDs are permanent.

Find your Twitch user ID: [https://www.streamweasels.com/tools/convert-twitch-username-to-user-id/](https://www.streamweasels.com/tools/convert-twitch-username-to-user-id/)

## Token refresh (optional)

Tokens from [Twitch Token Generator](https://twitchtokengenerator.com/) cannot auto-refresh — regenerate when expired.

For automatic refresh, create your own Twitch application at [Twitch Developer Console](https://dev.twitch.tv/console) and add to config:
```json
{
  "channels": {
    "twitch": {
      "clientSecret": "your_client_secret",
      "refreshToken": "your_refresh_token"
    }
  }
}
```

Bot automatically refreshes tokens before expiration and logs refresh events.

## Multi-account support

Use channels.twitch.accounts with per-account tokens.

Example (one bot account in two channels):
```json
{
  "channels": {
    "twitch": {
      "accounts": {
        "channel1": {
          "username": "openclaw",
          "accessToken": "oauth:abc123...",
          "clientId": "xyz789...",
          "channel": "vevisk"
        },
        "channel2": {
          "username": "openclaw",
          "accessToken": "oauth:def456...",
          "clientId": "uvw012...",
          "channel": "secondchannel"
        }
      }
    }
  }
}
```

Note: Each account needs its own token (one token per channel).

## Access control

### Role-based restrictions

```json
{
  "channels": {
    "twitch": {
      "accounts": {
        "default": {
          "allowedRoles": ["moderator", "vip"]
        }
      }
    }
  }
}
```

### Allowlist by User ID (most secure)

```json
{
  "channels": {
    "twitch": {
      "accounts": {
        "default": {
          "allowFrom": ["123456789", "987654321"]
        }
      }
    }
  }
}
```

### Disable @mention requirement

By default, requireMention is true. To disable and respond to all messages:
```json
{
  "channels": {
    "twitch": {
      "accounts": {
        "default": {
          "requireMention": false
        }
      }
    }
  }
}
```

## Troubleshooting

First, run diagnostic commands:
```bash
openclaw doctor
openclaw channels status --probe
```

### Bot doesn't respond to messages

Check access control: Ensure your user ID in allowFrom, or temporarily remove allowFrom and set allowedRoles: ["all"] to test.
Verify bot is in channel: Bot must join channel specified in channel.

### Token issues

"Failed to connect" or authentication errors:
- Verify accessToken is OAuth access token value (typically starts with oauth: prefix)
- Check token has chat:read and chat:write scopes
- If using token refresh, verify clientSecret and refreshToken set

### Token refresh not working

Check logs for refresh events:
```
Using env token source for mybot
Access token refreshed for user 123456 (expires in 14400s)
```

If you see "token refresh disabled (no refresh token)":
- Ensure clientSecret provided
- Ensure refreshToken provided

## Configuration

Account config:
- username: Bot username
- accessToken: OAuth access token with chat:read and chat:write
- clientId: Twitch Client ID
- channel: Channel to join (required)
- enabled: Enable this account (default: true)
- clientSecret: Optional, for automatic token refresh
- refreshToken: Optional, for automatic token refresh
- allowFrom: User ID allowlist
- allowedRoles: Role-based access control
- requireMention: Require @mention (default: true)

Provider options:
- channels.twitch.enabled: enable/disable channel startup
- channels.twitch.username: bot username (simplified single-account config)
- channels.twitch.accessToken: OAuth access token (simplified single-account config)
- channels.twitch.clientId: Twitch Client ID (simplified single-account config)
- channels.twitch.channel: Channel to join (simplified single-account config)
- channels.twitch.accounts.: Multi-account config

## Safety & ops

- Treat tokens like passwords — never commit to git
- Use automatic token refresh for long-running bots
- Use user ID allowlists instead of usernames for access control
- Monitor logs for token refresh events and connection status
- Scope tokens minimally — only request chat:read and chat:write
- If stuck: Restart gateway after confirming no other process owns session

## Limits

- 500 characters per message (auto-chunked at word boundaries)
- Markdown stripped before chunking
- No rate limiting (uses Twitch's built-in rate limits)
