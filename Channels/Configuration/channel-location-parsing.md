# Channel Location Parsing

Location parsing extracts geographic coordinates and place names from messages sent across different channels.

Status: Partially implemented. Some channels support location messages natively; parsing from text is planned.

## Native Location Support (Current)

### Channels with Native Location Messages

These channels support dedicated location message types:
- **LINE**: sends location messages with latitude/longitude/title/address
- **Google Chat**: supports location via geo URIs
- **Matrix**: supports geo URIs

When a user sends a location via these channels, OpenClaw receives:
- Latitude / Longitude
- Optional place name / address
- Optional altitude / accuracy

The location is surfaced to the agent as context:
```json
{
  "location": {
    "latitude": 51.5074,
    "longitude": -0.1278,
    "name": "Big Ben, London",
    "address": "Palace of Westminster, London, UK"
  }
}
```

### Channels Without Native Support

Other channels (Telegram, WhatsApp, Discord, etc.) receive location as:
- Text description ("I'm at home", "Meet me at the coffee shop")
- Google Maps / Apple Maps links
- Coordinates in text ("51.5074, -0.1278")
- Image attachments (maps, photos with location metadata)

## Text-Based Location Extraction (Planned)

Future updates will add ability to parse location mentions from plain text:
- Recognize place names (cities, landmarks, addresses)
- Extract coordinates from text
- Resolve place names to coordinates (geocoding)
- Build location context for agent

Example (planned):
```
User: "I'm at home in San Francisco"
→ Parsed: latitude=37.7749, longitude=-122.4194, place="San Francisco, CA"
```

## Using Location Context

When location data available, agent can:
- Provide location-specific recommendations
- Check weather at user's location
- Find nearby services
- Offer directions

Example agent action:
```json
{
  "action": "web_search",
  "query": "coffee shops near 51.5074, -0.1278"
}
```

## Privacy Considerations

Location is sensitive data. Consider:
- **Storage**: Disable history saving if locations logged
- **Context**: Don't include location in shared context
- **Tools**: Restrict web_search / browser tools if location shared
- **Opt-in**: Only request location when necessary

Config:
```json
{
  "channels": {
    "telegram": {
      "locationPrivacy": "hide",  // Don't log location in history
      "groups": {
        "*": {
          "tools": { "deny": ["web_search", "browser"] }  // Restrict tools in groups
        }
      }
    }
  }
}
```

## Configuration (Planned)

When implemented:
```json
{
  "location": {
    "parsing": {
      "enabled": true,
      "geocoding": true,  // Resolve place names to coordinates
      "includeInContext": true
    },
    "privacy": {
      "logLocations": false,  // Don't store in history
      "allowedChannels": ["telegram"]  // Only accept location from trusted channels
    }
  }
}
```

## Roadmap

Current implementation:
- ✅ Native location message support (LINE, Google Chat, Matrix)
- ✅ Surface location in inbound context

Planned:
- ⏳ Text-based location extraction
- ⏳ Geocoding (place names → coordinates)
- ⏳ Location privacy controls
- ⏳ Agent tools for location-based actions

Check GitHub issues for status: https://github.com/openclaw/openclaw
