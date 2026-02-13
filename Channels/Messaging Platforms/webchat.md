# WebChat (Gateway WebSocket UI)

Status: macOS/iOS SwiftUI chat UI talks directly to Gateway WebSocket.

## What it is

- Native chat UI for gateway (no embedded browser, no local static server)
- Uses same sessions and routing rules as other channels
- Deterministic routing: replies always go back to WebChat

## Quick start

- Start gateway
- Open WebChat UI (macOS/iOS app) or Control UI chat tab
- Ensure gateway auth configured (required by default, even on loopback)

## How it works (behavior)

- UI connects to Gateway WebSocket and uses chat.history, chat.send, and chat.inject
- chat.inject appends assistant note directly to transcript and broadcasts to UI (no agent run)
- History always fetched from gateway (no local file watching)
- If gateway unreachable, WebChat is read-only

## Remote use

- Remote mode tunnels gateway WebSocket over SSH/Tailscale
- No need to run separate WebChat server

## Configuration reference (WebChat)

Full configuration: [Configuration](/gateway/configuration)

Channel options:
- No dedicated webchat.* block. WebChat uses gateway endpoint + auth settings below.

Related global options:
- gateway.port, gateway.bind: WebSocket host/port
- gateway.auth.mode, gateway.auth.token, gateway.auth.password: WebSocket auth
- gateway.remote.url, gateway.remote.token, gateway.remote.password: remote gateway target
- session.*: session storage and main key defaults
