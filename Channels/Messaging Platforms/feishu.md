# Feishu (Lark) Bot

Feishu (Lark) is a team chat platform used for messaging and collaboration. This plugin connects OpenClaw to a Feishu/Lark bot using WebSocket event subscription (no public webhook URL needed).

## Plugin required

Install the Feishu plugin:
```bash
openclaw plugins install @openclaw/feishu
```

Local checkout (when running from git repo):
```bash
openclaw plugins install ./extensions/feishu
```

## Quickstart

### Method 1: onboarding wizard (recommended)

If you just installed OpenClaw:
```bash
openclaw onboard
```

The wizard guides you through:
- Creating a Feishu app and collecting credentials
- Configuring app credentials in OpenClaw
- Starting the gateway

After configuration, check status:
```bash
openclaw gateway status
openclaw logs --follow
```

### Method 2: CLI setup

If you already completed initial install:
```bash
openclaw channels add
```

Choose Feishu, enter App ID and App Secret.

After configuration:
```bash
openclaw gateway status
openclaw gateway restart
openclaw logs --follow
```

## Step 1: Create a Feishu app

### 1. Open Feishu Open Platform

Visit [Feishu Open Platform](https://open.feishu.cn/app) and sign in.
Lark (global) tenants use [https://open.larksuite.com/app](https://open.larksuite.com/app) and set domain: "lark" in config.

### 2. Create an app

- Click "Create enterprise app"
- Fill in app name + description
- Choose app icon

### 3. Copy credentials

From Credentials & Basic Info:
- App ID (format: cli_xxx)
- App Secret (keep private!)

### 4. Configure permissions

On Permissions, click "Batch import" and paste the required scopes (see full docs for scope list).

### 5. Enable bot capability

In App Capability > Bot:
- Enable bot capability
- Set bot name

### 6. Configure event subscription

⚠️ **Before setting event subscription, ensure:**
- You already ran `openclaw channels add` for Feishu
- Gateway is running (`openclaw gateway status`)

In Event Subscription:
- Choose "Use long connection to receive events" (WebSocket)
- Add event: im.message.receive_v1

⚠️ If gateway not running, long-connection setup may fail.

### 7. Publish the app

- Create version in Version Management & Release
- Submit for review and publish
- Wait for admin approval (enterprise apps usually auto-approve)

## Step 2: Configure OpenClaw

### Via CLI
```bash
openclaw channels add
```

Choose Feishu and paste your App ID + App Secret.

### Via config file

Edit ~/.openclaw/openclaw.json:
```json
{
  "channels": {
    "feishu": {
      "enabled": true,
      "dmPolicy": "pairing",
      "accounts": {
        "main": {
          "appId": "cli_xxx",
          "appSecret": "xxx",
          "botName": "My AI assistant"
        }
      }
    }
  }
}
```

### Via environment variables

```bash
export FEISHU_APP_ID="cli_xxx"
export FEISHU_APP_SECRET="xxx"
```

### Lark (global) domain

If your tenant is on Lark (international), set domain to "lark":
```json
{
  "channels": {
    "feishu": {
      "domain": "lark",
      "accounts": {
        "main": {
          "appId": "cli_xxx",
          "appSecret": "xxx"
        }
      }
    }
  }
}
```

## Step 3: Start + test

### 1. Start the gateway
```bash
openclaw gateway
```

### 2. Send a test message

In Feishu, find your bot and send a message.

### 3. Approve pairing

Bot replies with pairing code by default. Approve it:
```bash
openclaw pairing approve feishu <CODE>
```

After approval, you can chat normally.

## Overview

- Feishu bot channel managed by the gateway
- Deterministic routing: replies always return to Feishu
- Session isolation: DMs share main session; groups isolated
- WebSocket connection: long connection via Feishu SDK, no public URL needed

## Access control

### Direct messages

- Default: dmPolicy: "pairing" (unknown users get pairing code)
- Approve pairing: `openclaw pairing list feishu` / `openclaw pairing approve feishu`
- Allowlist mode: set channels.feishu.allowFrom with allowed Open IDs

### Group chats

1. **Group policy** (channels.feishu.groupPolicy):
   - "open" = allow everyone in groups (default)
   - "allowlist" = only allow groupAllowFrom
   - "disabled" = disable group messages

2. **Mention requirement** (channels.feishu.groups.<chat_id>.requireMention):
   - true = require @mention (default)
   - false = respond without mentions

## Group configuration examples

### Allow all groups, require @mention (default)
```json
{
  "channels": {
    "feishu": {
      "groupPolicy": "open"
    }
  }
}
```

### Allow all groups, no @mention required
```json
{
  "channels": {
    "feishu": {
      "groups": {
        "oc_xxx": { "requireMention": false }
      }
    }
  }
}
```

### Allow specific users in groups only
```json
{
  "channels": {
    "feishu": {
      "groupPolicy": "allowlist",
      "groupAllowFrom": ["ou_xxx", "ou_yyy"]
    }
  }
}
```

## Get group/user IDs

### Group IDs (chat_id)

Group IDs look like oc_xxx.

**Method 1 (recommended):**
- Start gateway and @mention bot in group
- Run `openclaw logs --follow` and look for chat_id

**Method 2:**
Use Feishu API debugger to list group chats.

### User IDs (open_id)

User IDs look like ou_xxx.

**Method 1 (recommended):**
- Start gateway and DM bot
- Run `openclaw logs --follow` and look for open_id

**Method 2:**
Check pairing requests: `openclaw pairing list feishu`

## Common commands

| Command | Description |
|---------|-------------|
| /status | Show bot status |
| /reset | Reset the session |
| /model | Show/switch model |

Note: Feishu doesn't support native command menus yet, so commands sent as text.

## Troubleshooting

### Bot does not respond in group chats

- Ensure bot is added to group
- Ensure you @mention bot (default behavior)
- Check groupPolicy is not set to "disabled"
- Check logs: `openclaw logs --follow`

### Bot does not receive messages

- Ensure app is published and approved
- Ensure event subscription includes im.message.receive_v1
- Ensure long connection is enabled
- Ensure app permissions are complete
- Ensure gateway is running: `openclaw gateway status`
- Check logs: `openclaw logs --follow`

### App Secret leak

- Reset App Secret in Feishu Open Platform
- Update App Secret in your config
- Restart gateway

### Message send failures

- Ensure app has im:message:send_as_bot permission
- Ensure app is published
- Check logs for detailed errors

## Advanced configuration

### Multiple accounts

```json
{
  "channels": {
    "feishu": {
      "accounts": {
        "main": {
          "appId": "cli_xxx",
          "appSecret": "xxx",
          "botName": "Primary bot"
        },
        "backup": {
          "appId": "cli_yyy",
          "appSecret": "yyy",
          "botName": "Backup bot",
          "enabled": false
        }
      }
    }
  }
}
```

### Message limits

- textChunkLimit: outbound text chunk size (default: 2000 chars)
- mediaMaxMb: media upload/download limit (default: 30MB)

### Streaming

Feishu supports streaming replies via interactive cards:
```json
{
  "channels": {
    "feishu": {
      "streaming": true,
      "blockStreaming": true
    }
  }
}
```

Set streaming: false to wait for full reply before sending.

## Configuration reference

Full configuration: [Configuration](/gateway/configuration)

Key options:
- channels.feishu.enabled: enable/disable channel
- channels.feishu.domain: API domain (feishu or lark)
- channels.feishu.accounts.<id>.appId: App ID
- channels.feishu.accounts.<id>.appSecret: App Secret
- channels.feishu.dmPolicy: DM policy (pairing, allowlist, open, disabled)
- channels.feishu.allowFrom: DM allowlist (open_id list)
- channels.feishu.groupPolicy: Group policy (open, allowlist, disabled)
- channels.feishu.groupAllowFrom: Group allowlist
- channels.feishu.groups.<chat_id>.requireMention: Require @mention
- channels.feishu.textChunkLimit: Message chunk size (default: 2000)
- channels.feishu.mediaMaxMb: Media size limit (default: 30)
- channels.feishu.streaming: Enable streaming card output (default: true)
- channels.feishu.blockStreaming: Enable block streaming (default: true)

## Supported message types

### Receive
- ✅ Text
- ✅ Rich text (post)
- ✅ Images
- ✅ Files
- ✅ Audio
- ✅ Video
- ✅ Stickers

### Send
- ✅ Text
- ✅ Images
- ✅ Files
- ✅ Audio
- ⚠️ Rich text (partial support)
