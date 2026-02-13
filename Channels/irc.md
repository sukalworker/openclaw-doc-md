# IRC

Use IRC when you want OpenClaw in classic channels (#room) and direct messages.
IRC ships as an extension plugin, but is configured in the main config under channels.irc.

## Quick start

Enable IRC config:
```json
{
  "channels": {
    "irc": {
      "enabled": true,
      "host": "irc.libera.chat",
      "port": 6697,
      "tls": true,
      "nick": "openclaw-bot",
      "channels": ["#openclaw"]
    }
  }
}
```

Restart gateway: `openclaw gateway run`

## Security defaults

- channels.irc.dmPolicy defaults to "pairing"
- channels.irc.groupPolicy defaults to "allowlist"
- With groupPolicy="allowlist", set channels.irc.groups to define allowed channels
- Use TLS (channels.irc.tls=true) unless intentionally accepting plaintext

## Access control

Two separate "gates":
- Channel access (groupPolicy + groups): whether bot accepts messages from channel
- Sender access (groupAllowFrom / per-channel groups[].allowFrom): who can trigger bot

Config keys:
- DM allowlist: channels.irc.allowFrom
- Group sender allowlist: channels.irc.groupAllowFrom
- Per-channel controls: channels.irc.groups["#channel"]
- channels.irc.groupPolicy="open" allows unconfigured channels (still mention-gated by default)

Allowlist entries use nick or nick!user@host forms.

### Common gotcha: allowFrom is for DMs, not channels

If you see logs like "irc: drop group sender alice!ident@host (policy=allowlist)", sender not allowed for group messages.

Fix by either:
- Setting channels.irc.groupAllowFrom (global)
- Setting per-channel sender allowlist: channels.irc.groups["#channel"].allowFrom

Example (allow anyone in #tuirc-dev):
```json
{
  "channels": {
    "irc": {
      "groupPolicy": "allowlist",
      "groups": {
        "#tuirc-dev": { "allowFrom": ["*"] }
      }
    }
  }
}
```

## Reply triggering (mentions)

Default mention-gating in group contexts. To disable for a channel:
```json
{
  "channels": {
    "irc": {
      "groupPolicy": "allowlist",
      "groups": {
        "#tuirc-dev": {
          "requireMention": false,
          "allowFrom": ["*"]
        }
      }
    }
  }
}
```

Or allow all IRC channels without mention requirement:
```json
{
  "channels": {
    "irc": {
      "groupPolicy": "open",
      "groups": {
        "*": { "requireMention": false, "allowFrom": ["*"] }
      }
    }
  }
}
```

## Security note (recommended for public channels)

If allowing allowFrom: ["*"] in public channel, restrict tools:
```json
{
  "channels": {
    "irc": {
      "groups": {
        "#tuirc-dev": {
          "allowFrom": ["*"],
          "tools": {
            "deny": ["group:runtime", "group:fs", "gateway", "nodes", "cron", "browser"]
          }
        }
      }
    }
  }
}
```

Or different tools per sender (owner gets more power):
```json
{
  "channels": {
    "irc": {
      "groups": {
        "#tuirc-dev": {
          "allowFrom": ["*"],
          "toolsBySender": {
            "*": {
              "deny": ["group:runtime", "group:fs", "gateway", "nodes", "cron", "browser"]
            },
            "eigen": {
              "deny": ["gateway", "nodes", "cron"]
            }
          }
        }
      }
    }
  }
}
```

Notes:
- toolsBySender keys can be nick (e.g. "eigen") or full hostmask (e.g. "eigen@host") for stronger matching
- First matching sender policy wins; "*" is wildcard fallback

## NickServ

To identify with NickServ after connect:
```json
{
  "channels": {
    "irc": {
      "nickserv": {
        "enabled": true,
        "service": "NickServ",
        "password": "your-nickserv-password"
      }
    }
  }
}
```

Optional registration on connect:
```json
{
  "channels": {
    "irc": {
      "nickserv": {
        "register": true,
        "registerEmail": "user@example.com"
      }
    }
  }
}
```

## Environment variables

Default account supports:
- IRC_HOST, IRC_PORT, IRC_TLS, IRC_NICK, IRC_USERNAME, IRC_REALNAME
- IRC_PASSWORD, IRC_CHANNELS (comma-separated)
- IRC_NICKSERV_PASSWORD, IRC_NICKSERV_REGISTER_EMAIL

## Troubleshooting

- Bot connects but never replies: verify groups config and mention-gating (missing-mention drops). Set requireMention:false if no pings wanted.
- Login fails: verify nick availability and server password
- TLS fails: verify host/port and certificate setup
