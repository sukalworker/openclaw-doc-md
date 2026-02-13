# Microsoft Teams (plugin)

"Abandon all hope, ye who enter here."

Status: text + DM attachments supported; channel/group file sending requires sharePointSiteId + Graph permissions. Polls sent via Adaptive Cards.

## Plugin required

Microsoft Teams ships as plugin, not bundled with core.

Breaking change (2026.1.15): MS Teams moved out of core. Must install plugin if using.

Install via CLI (npm registry):
```bash
openclaw plugins install @openclaw/msteams
```

Local checkout (git repo):
```bash
openclaw plugins install ./extensions/msteams
```

If you choose Teams during configure/onboarding and git checkout detected, OpenClaw will offer local install path automatically.

Details: [Plugins](/tools/plugin)

## Quick setup (beginner)

- Install Microsoft Teams plugin
- Create Azure Bot (App ID + client secret + tenant ID)
- Configure OpenClaw with those credentials
- Expose /api/messages (port 3978 by default) via public URL or tunnel
- Install Teams app package and start gateway

Minimal config:
```json
{
  "channels": {
    "msteams": {
      "enabled": true,
      "appId": "<APP_ID>",
      "appPassword": "<APP_PASSWORD>",
      "tenantId": "<TENANT_ID>",
      "webhook": { "port": 3978, "path": "/api/messages" }
    }
  }
}
```

Note: group chats blocked by default (channels.msteams.groupPolicy: "allowlist"). To allow group replies, set channels.msteams.groupAllowFrom (or use groupPolicy: "open" to allow any member, mention-gated).

## Goals

- Talk to OpenClaw via Teams DMs, group chats, or channels
- Keep routing deterministic: replies always go back to channel they arrived on
- Default to safe channel behavior (mentions required unless configured otherwise)

## Config writes

Microsoft Teams allows config updates via /config set|unset (requires commands.config: true).

Disable with:
```json
{
  "channels": { "msteams": { "configWrites": false } }
}
```

## Access control (DMs + groups)

### DM access

- Default: channels.msteams.dmPolicy = "pairing" (unknown senders ignored until approved)
- channels.msteams.allowFrom accepts AAD object IDs, UPNs, or display names (wizard resolves names via Graph when credentials allow)

### Group access

- Default: channels.msteams.groupPolicy = "allowlist" (blocked unless you add groupAllowFrom)
- channels.msteams.groupAllowFrom controls which senders can trigger in group chats/channels (fallback to channels.msteams.allowFrom)
- Set groupPolicy: "open" to allow any member (still mention-gated by default)
- Set groupPolicy: "disabled" to allow no channels

Example:
```json
{
  "channels": {
    "msteams": {
      "groupPolicy": "allowlist",
      "groupAllowFrom": ["user@example.com"]
    }
  }
}
```

### Teams + channel allowlist

Scope group/channel replies by listing teams and channels under channels.msteams.teams.
Keys can be team IDs or names; channel keys can be conversation IDs or names.

When groupPolicy="allowlist" and teams allowlist present, only listed teams/channels accepted (mention-gated).

Configure wizard accepts Team/Channel entries and stores them. On startup, OpenClaw resolves team/channel and user allowlist names to IDs (when Graph permissions allow); unresolved entries kept as typed.

Example:
```json
{
  "channels": {
    "msteams": {
      "groupPolicy": "allowlist",
      "teams": {
        "My Team": {
          "channels": {
            "General": { "requireMention": true }
          }
        }
      }
    }
  }
}
```

## How it works

- Install Microsoft Teams plugin
- Create Azure Bot (App ID + secret + tenant ID)
- Build Teams app package referencing bot and including RSC permissions
- Upload/install Teams app into team (or personal scope for DMs)
- Configure msteams in ~/.openclaw/openclaw.json (or env vars) and start gateway
- Gateway listens for Bot Framework webhook traffic on /api/messages by default

## Azure Bot Setup (Prerequisites)

Before configuring OpenClaw, create Azure Bot resource.

### Step 1: Create Azure Bot

Go to [Create Azure Bot](https://portal.azure.com/#create/Microsoft.AzureBot)

Fill in Basics tab:
| Field | Value |
|-------|-------|
| Bot handle | Your bot name (must be unique), e.g., openclaw-msteams |
| Subscription | Select your Azure subscription |
| Resource group | Create new or use existing |
| Pricing tier | Free for dev/testing |
| Type of App | Single Tenant (recommended) |
| Creation type | Create new Microsoft App ID |

Deprecation notice: Multi-tenant bots deprecated after 2025-07-31. Use Single Tenant for new bots.

Click Review + create → Create (wait 1-2 minutes)

### Step 2: Get Credentials

Go to Azure Bot resource → Configuration

- Copy Microsoft App ID → this is your appId
- Click Manage Password → go to App Registration
- Under Certificates & secrets → New client secret → copy Value → this is your appPassword
- Go to Overview → copy Directory (tenant) ID → this is your tenantId

### Step 3: Configure Messaging Endpoint

In Azure Bot → Configuration

Set Messaging endpoint to your webhook URL:

Production: `https://your-domain.com/api/messages`

Local dev: Use a tunnel (see [Local Development](#local-development-tunneling) below)

### Step 4: Enable Teams Channel

In Azure Bot → Channels

- Click Microsoft Teams → Configure → Save
- Accept the Terms of Service

## Local Development (Tunneling)

Teams can't reach localhost. Use a tunnel for local development:

Option A: ngrok
```bash
ngrok http 3978
# Copy the https URL, e.g., https://abc123.ngrok.io
# Set messaging endpoint to: https://abc123.ngrok.io/api/messages
```

Option B: Tailscale Funnel
```bash
tailscale funnel 3978
# Use your Tailscale funnel URL as the messaging endpoint
```

## Teams Developer Portal (Alternative)

Instead of manually creating manifest ZIP, use [Teams Developer Portal](https://dev.teams.microsoft.com/apps):
- Click + New app
- Fill in basic info (name, description, developer info)
- Go to App features → Bot
- Select "Enter a bot ID manually" and paste your Azure Bot App ID
- Check scopes: Personal, Team, Group Chat
- Click Distribute → Download app package
- In Teams: Apps → Manage your apps → Upload a custom app → select ZIP

Often easier than hand-editing JSON manifests.

## Testing the Bot

Option A: Azure Web Chat (verify webhook first)
- In Azure Portal → your Azure Bot resource → Test in Web Chat
- Send message — you should see response
- Confirms webhook works before Teams setup

Option B: Teams (after app installation)
- Install Teams app (sideload or org catalog)
- Find bot in Teams and send DM
- Check gateway logs for incoming activity

## Setup (minimal text-only)

- Install Microsoft Teams plugin: `openclaw plugins install @openclaw/msteams` or local checkout
- Bot registration: Create Azure Bot (see above), note App ID, Client secret, Tenant ID
- Teams app manifest: Include bot entry with botId = <App ID>
  - Scopes: personal, team, groupChat
  - supportsFiles: true (required for personal scope file handling)
  - Add RSC permissions (see extended docs)
  - Create icons: outline.png (32x32) and color.png (192x192)
  - Zip all: manifest.json, outline.png, color.png
- Configure OpenClaw:
  ```json
  {
    "msteams": {
      "enabled": true,
      "appId": "<APP_ID>",
      "appPassword": "<APP_PASSWORD>",
      "tenantId": "<TENANT_ID>",
      "webhook": { "port": 3978, "path": "/api/messages" }
    }
  }
  ```
  Can also use env vars: MSTEAMS_APP_ID, MSTEAMS_APP_PASSWORD, MSTEAMS_TENANT_ID
- Bot endpoint: Set Azure Bot Messaging Endpoint to: `https://<domain>:3978/api/messages` (or your chosen path/port)
- Run gateway: Teams channel starts automatically

## History context

- channels.msteams.historyLimit controls recent channel/group messages wrapped into prompt
- Falls back to messages.groupChat.historyLimit (default 50, set 0 to disable)
- DM history: channels.msteams.dmHistoryLimit (user turns)
- Per-user overrides: channels.msteams.dms["<id>"].historyLimit

## Configuration

Key settings (see /gateway/configuration for shared channel patterns):
- channels.msteams.enabled: enable/disable channel
- channels.msteams.appId, channels.msteams.appPassword, channels.msteams.tenantId: bot credentials
- channels.msteams.webhook.port (default 3978)
- channels.msteams.webhook.path (default /api/messages)
- channels.msteams.dmPolicy: pairing | allowlist | open | disabled (default: pairing)
- channels.msteams.allowFrom: allowlist for DMs (AAD object IDs, UPNs, or display names)
- channels.msteams.textChunkLimit: outbound text chunk size
- channels.msteams.chunkMode: length (default) or newline
- channels.msteams.mediaAllowHosts: allowlist for inbound attachment hosts (defaults to Microsoft/Teams domains)
- channels.msteams.mediaAuthAllowHosts: allowlist for Authorization headers on media retries
- channels.msteams.requireMention: require @mention in channels/groups (default true)
- channels.msteams.replyStyle: thread | top-level
- channels.msteams.teams.*.replyStyle: per-team override
- channels.msteams.teams.*.requireMention: per-team override
- channels.msteams.teams.*.tools: per-team tool policy overrides
- channels.msteams.teams.*.toolsBySender: per-team per-sender tool policy overrides
- channels.msteams.teams.*.channels.*.replyStyle: per-channel override
- channels.msteams.teams.*.channels.*.requireMention: per-channel override
- channels.msteams.teams.*.channels.*.tools: per-channel tool policy overrides
- channels.msteams.teams.*.channels.*.toolsBySender: per-channel per-sender tool policy overrides
- channels.msteams.sharePointSiteId: SharePoint site ID for file uploads in group chats/channels (see [Sending files in group chats](#sending-files-in-group-chats))

## Troubleshooting

Run diagnostic ladder:
```bash
openclaw status
openclaw gateway status
openclaw logs --follow
openclaw doctor
openclaw channels status --probe
```

Common issues:
- Images not showing in channels: Graph permissions or admin consent missing. Reinstall Teams app, fully quit/reopen Teams.
- No responses in channel: mentions required by default; set channels.msteams.requireMention=false or configure per team/channel
- Version mismatch: remove + re-add app, fully quit Teams to refresh
- 401 Unauthorized from webhook: Expected when testing manually without Azure JWT — means endpoint reachable but auth failed. Use Azure Web Chat.

Manifest upload errors:
- "Icon file cannot be empty": manifest references 0-byte icon files. Create valid PNGs (32x32 outline.png, 192x192 color.png)
- "webApplicationInfo.Id already in use": app still installed elsewhere. Find and uninstall, or wait 5-10 minutes
- "Something went wrong": Upload via [https://admin.teams.microsoft.com](https://admin.teams.microsoft.com), open DevTools (F12) → Network tab, check response body
- Sideload failing: Try "Upload an app to your org's app catalog" instead of "Upload a custom app"

RSC permissions not working:
- Verify webApplicationInfo.id matches bot's App ID exactly
- Re-upload app and reinstall in team/chat
- Check if org admin blocked RSC permissions
- Confirm right scope: ChannelMessage.Read.Group for teams, ChatMessage.Read.Chat for group chats

## References

- [Create Azure Bot](https://learn.microsoft.com/en-us/azure/bot-service/bot-service-quickstart-registration)
- [Teams Developer Portal](https://dev.teams.microsoft.com/apps)
- [Teams app manifest schema](https://learn.microsoft.com/en-us/microsoftteams/platform/resources/schema/manifest-schema)
- [Receive channel messages with RSC](https://learn.microsoft.com/en-us/microsoftteams/platform/bots/how-to/conversations/channel-messages-with-rsc)
- [RSC permissions reference](https://learn.microsoft.com/en-us/microsoftteams/platform/graph-api/rsc/resource-specific-consent)
- [Teams bot file handling](https://learn.microsoft.com/en-us/microsoftteams/platform/bots/how-to/bots-filesv4)
- [Proactive messaging](https://learn.microsoft.com/en-us/microsoftteams/platform/bots/how-to/conversations/send-proactive-messages)
