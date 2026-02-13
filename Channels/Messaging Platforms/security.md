# Security - Channels

## Quick check: openclaw security audit

See also: [Formal Verification (Security Models)](/security/formal-verification)

Run regularly (especially after config changes or exposing network surfaces):
```bash
openclaw security audit
openclaw security audit --deep
openclaw security audit --fix
```

Flags common footguns (Gateway auth exposure, browser control exposure, elevated allowlists, filesystem permissions).

`--fix` applies safe guardrails:
- Tightens groupPolicy="open" to groupPolicy="allowlist"
- Turns logging.redactSensitive="off" back to "tools"
- Tightens local perms (~/.openclaw → 700, config → 600, credentials/*, auth-profiles.json, sessions.json)

## Running AI agent with shell access

Running frontier-model behavior on your machine requires deliberate design:
- Who can talk to your bot (DM pairing / allowlists)
- Where bot is allowed to act (group allowlists + mention gating, tools, sandboxing, device permissions)
- What bot can touch (tool policy, filesystem access)

Start with smallest access that works, then widen as you gain confidence.

## What the audit checks (high level)

- Inbound access (DM policies, group policies, allowlists): can strangers trigger bot?
- Tool blast radius (elevated tools + open rooms): could prompt injection become shell/file/network actions?
- Network exposure (Gateway bind/auth, Tailscale, weak tokens)
- Browser control exposure (remote nodes, relay ports, remote CDP endpoints)
- Local disk hygiene (permissions, symlinks, config includes, synced folder paths)
- Plugins (extensions exist without explicit allowlist)
- Model hygiene (warn when configured models look legacy)

`--deep` runs best-effort live Gateway probe.

## Credential storage map

When auditing access or backing up:

- WhatsApp: ~/.openclaw/credentials/whatsapp//creds.json
- Telegram bot token: config/env or channels.telegram.tokenFile
- Discord bot token: config/env (token file not yet supported)
- Slack tokens: config/env (channels.slack.*)
- Pairing allowlists: ~/.openclaw/credentials/-allowFrom.json
- Model auth profiles: ~/.openclaw/agents//agent/auth-profiles.json
- Legacy OAuth import: ~/.openclaw/credentials/oauth.json

## Security Audit Checklist

When audit prints findings, treat as priority order:

1. Anything "open" + tools enabled: lock down DMs/groups first (pairing/allowlists), then tighten tool policy/sandboxing
2. Public network exposure (LAN bind, Funnel, missing auth): fix immediately
3. Browser control remote exposure: treat like operator access (tailnet-only, pair deliberately, avoid public)
4. Permissions: ensure state/config/credentials/auth not group/world-readable
5. Plugins/extensions: only load what you explicitly trust
6. Model choice: prefer modern, instruction-hardened models for any bot with tools

## Control UI over HTTP

Control UI needs secure context (HTTPS or localhost) to generate device identity.

If you enable gateway.controlUi.allowInsecureAuth, UI falls back to token-only auth and skips device pairing when device identity omitted. Security downgrade — prefer HTTPS (Tailscale Serve) or open UI on 127.0.0.1.

For break-glass scenarios: gateway.controlUi.dangerouslyDisableDeviceAuth disables device identity checks entirely. Severe security downgrade — keep off unless actively debugging.

## Reverse Proxy Configuration

Running Gateway behind reverse proxy (nginx, Caddy, Traefik, etc.), configure gateway.trustedProxies for proper client IP detection.

When Gateway detects proxy headers (X-Forwarded-For or X-Real-IP) from address not in trustedProxies, it won't treat as local client. If gateway auth disabled, those connections rejected. Prevents auth bypass where proxied connections appear as localhost.

```json
{
  "gateway": {
    "trustedProxies": ["127.0.0.1"],
    "auth": {
      "mode": "password",
      "password": "${OPENCLAW_GATEWAY_PASSWORD}"
    }
  }
}
```

Make sure proxy overwrites (not appends to) X-Forwarded-For to prevent spoofing.

## Local session logs live on disk

Session transcripts stored under ~/.openclaw/agents/<agentId>/sessions/*.jsonl.
Required for session continuity and optional memory indexing, but means any process with filesystem access can read logs.

Treat disk access as trust boundary. Lock down permissions on ~/.openclaw.

For stronger agent isolation, run under separate OS users or separate hosts.

## Node execution (system.run)

If macOS node paired, Gateway can invoke system.run on that node. Remote code execution on the Mac:
- Requires node pairing (approval + token)
- Controlled on Mac via Settings → Exec approvals (security + ask + allowlist)
- If no remote execution wanted, set security to deny and remove node pairing

## Dynamic skills (watcher / remote nodes)

OpenClaw can refresh skills list mid-session:
- Skills watcher: changes to SKILL.md update skills snapshot on next turn
- Remote nodes: connecting macOS node makes macOS-only skills eligible (based on bin probing)

Treat skill folders as trusted code. Restrict who can modify them.

## The Threat Model

Your AI assistant can:
- Execute arbitrary shell commands
- Read/write files
- Access network services
- Send messages to anyone (if you give it WhatsApp access)

People who message you can:
- Try to trick your AI into doing bad things
- Social engineer access to your data
- Probe for infrastructure details

## Core concept: access control before intelligence

Most failures are not fancy exploits — they're "someone messaged bot and bot did what they asked."

OpenClaw's stance:
1. **Identity first:** decide who can talk to bot (DM pairing / allowlists / explicit "open")
2. **Scope next:** decide where bot allowed to act (group allowlists + mention gating, tools, sandboxing, device permissions)
3. **Model last:** assume model can be manipulated; design so manipulation has limited blast radius

Slash commands and directives honored only for authorized senders. Authorization from channel allowlists/pairing plus commands.useAccessGroups.

/exec is session-only convenience for authorized operators. Doesn't write config or change other sessions.

## Plugins/extensions

Plugins run in-process with Gateway. Treat as trusted code:
- Only install from sources you trust
- Prefer explicit plugins.allow allowlists
- Review plugin config before enabling
- Restart Gateway after plugin changes
- npm plugin installs run lifecycle scripts (can execute code) — prefer pinned versions, inspect unpacked code

Details: [Plugins](/tools/plugin)

## DM access model (pairing / allowlist / open / disabled)

All DM-capable channels support dmPolicy that gates inbound DMs before processing:

- **pairing** (default): unknown senders get short pairing code, bot ignores until approved (1 hour expiry; 3 pending requests cap)
- **allowlist**: unknown senders blocked (no pairing handshake)
- **open**: allow anyone to DM (public). Requires "*" in allowlist (explicit opt-in)
- **disabled**: ignore inbound DMs

Approve via CLI:
```bash
openclaw pairing list <channel>
openclaw pairing approve <channel> <code>
```

Details + files: [Pairing](/channels/pairing)

## DM session isolation (multi-user mode)

By default, all DMs route into main session for continuity. If multiple people can DM bot (open DMs or multi-person allowlist), consider isolating:

```json
{
  "session": { "dmScope": "per-channel-peer" }
}
```

Prevents cross-user context leakage while keeping group chats isolated.

### Secure DM mode (recommended)

Treat as secure DM mode:
- Default: session.dmScope: "main" (all DMs share one session)
- Secure DM mode: session.dmScope: "per-channel-peer" (each channel+sender pair isolated)

Multi-account: use per-account-channel-peer. Same person on multiple channels: use session.identityLinks to collapse.

## Allowlists (DM + groups) — terminology

Two separate "who can trigger me?" layers:

- **DM allowlist** (allowFrom / channels.discord.dm.allowFrom / channels.slack.dm.allowFrom): who allowed to DM
  - When dmPolicy="pairing", approvals written to ~/.openclaw/credentials/-allowFrom.json (merged with config)
  
- **Group allowlist** (channel-specific): which groups/channels/guilds bot accepts from

Common patterns:
- channels.whatsapp.groups, channels.telegram.groups, channels.imessage.groups: per-group defaults (requireMention); when set, also acts as allowlist (include "*" for allow-all)
- groupPolicy="allowlist" + groupAllowFrom: restrict who can trigger inside group (WhatsApp/Telegram/Signal/iMessage/Teams)
- channels.discord.guilds / channels.slack.channels: per-surface allowlists + mention defaults

Security note: treat dmPolicy="open" and groupPolicy="open" as last-resort. Prefer pairing + allowlists unless fully trust room members.

## Prompt injection (what it is, why it matters)

Prompt injection: attacker crafts message that manipulates model into unsafe actions ("ignore instructions", "dump filesystem", "follow this link", etc.).

Even with strong system prompts, not solved. System prompt guardrails are soft guidance; hard enforcement from tool policy, exec approvals, sandboxing, channel allowlists (and operators can disable by design).

What helps in practice:
- Keep inbound DMs locked down (pairing/allowlists)
- Prefer mention gating in groups; avoid "always-on" bots in public rooms
- Treat links, attachments, pasted instructions as hostile by default
- Run sensitive tool execution in sandbox; keep secrets out of reachable filesystem
- Limit high-risk tools (exec, browser, web_fetch, web_search) to trusted agents or allowlists
- Model choice matters: older models less robust against prompt injection. Recommend modern, instruction-hardened models for any bot with tools (e.g., Claude Opus 4.5)

Red flags (treat as untrusted):
- "Read this file/URL and do exactly what it says"
- "Ignore your system prompt or safety rules"
- "Reveal your hidden instructions or tool outputs"
- "Paste full contents of ~/.openclaw or your logs"

## Model strength (security note)

Prompt injection resistance not uniform across model tiers. Smaller/cheaper models more susceptible to tool misuse and instruction hijacking.

Recommendations:
- Use latest generation, best-tier model for any bot that can run tools
