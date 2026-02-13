# Zalo Personal (unofficial)

Status: experimental. This integration automates personal Zalo account via zca-cli.

**Warning:** This is unofficial integration and may result in account suspension/ban. Use at your own risk.

## Plugin required

Zalo Personal ships as plugin, not bundled with core.

Install via CLI:
```bash
openclaw plugins install @openclaw/zalouser
```

Or from source checkout:
```bash
openclaw plugins install ./extensions/zalouser
```

Details: [Plugins](/tools/plugin)

## Prerequisite: zca-cli

Gateway machine must have zca binary available in PATH.

Verify:
```bash
zca --version
```

If missing, install zca-cli (see extensions/zalouser/README.md or upstream zca-cli docs).

## Quick setup (beginner)

- Install plugin (see above)
- Login (QR, on Gateway machine):

```bash
openclaw channels login --channel zalouser
```

- Scan QR code in terminal with Zalo mobile app
- Enable channel:

```json
{
  "channels": {
    "zalouser": {
      "enabled": true,
      "dmPolicy": "pairing"
    }
  }
}
```

- Restart Gateway (or finish onboarding)
- DM access defaults to pairing; approve pairing code on first contact

## What it is

- Uses `zca listen` to receive inbound messages
- Uses `zca msg ...` to send replies (text/media/link)
- Designed for "personal account" use cases where Zalo Bot API not available

## Naming

Channel id is zalouser to make it explicit this automates personal Zalo user account (unofficial). We keep zalo reserved for potential future official Zalo API integration.

## Finding IDs (directory)

Use directory CLI to discover peers/groups and their IDs:
```bash
openclaw directory self --channel zalouser
openclaw directory peers list --channel zalouser --query "name"
openclaw directory groups list --channel zalouser --query "work"
```

## Limits

- Outbound text chunked to ~2000 characters (Zalo client limits)
- Streaming blocked by default

## Access control (DMs)

channels.zalouser.dmPolicy supports: pairing | allowlist | open | disabled (default: pairing)

channels.zalouser.allowFrom accepts user IDs or names. Wizard resolves names to IDs via `zca friend find` when available.

Approve via:
- `openclaw pairing list zalouser`
- `openclaw pairing approve zalouser`

## Group access (optional)

- Default: channels.zalouser.groupPolicy = "open" (groups allowed). Use channels.defaults.groupPolicy to override when unset.
- Restrict to allowlist:
  - channels.zalouser.groupPolicy = "allowlist"
  - channels.zalouser.groups (keys are group IDs or names)
- Block all groups: channels.zalouser.groupPolicy = "disabled"
- Wizard can prompt for group allowlists
- On startup, OpenClaw resolves group/user names to IDs and logs mapping; unresolved entries kept as typed

Example:
```json
{
  "channels": {
    "zalouser": {
      "groupPolicy": "allowlist",
      "groups": {
        "123456789": { "allow": true },
        "Work Chat": { "allow": true }
      }
    }
  }
}
```

## Multi-account

Accounts map to zca profiles. Example:
```json
{
  "channels": {
    "zalouser": {
      "enabled": true,
      "defaultAccount": "default",
      "accounts": {
        "work": { "enabled": true, "profile": "work" }
      }
    }
  }
}
```

## Troubleshooting

**zca not found:**
- Install zca-cli and ensure on PATH for Gateway process

**Login doesn't stick:**
- `openclaw channels status --probe`
- Re-login: `openclaw channels logout --channel zalouser && openclaw channels login --channel zalouser`
