# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

A collection of browser-playable games implemented as self-contained single HTML files (no build step, no dependencies, no server required). Open any file directly in a browser with `open <file>.html`.

## Files

- `tictactoe.html` — 2-player Tic Tac Toe with score tracking
  - Generated with prompt: *"make tic tac toe that i can play on the web"*
- `go.html` — 19×19 Go with full Japanese rules scoring (territory counting, dead stone marking, komi), supports online multiplayer
  - Generated with prompt: *"make a 19 by 19 Go game that i can play on web. make it auto count the result with Japanese Go rules."*
  - Multiplayer added with prompt: *"Upgrade go.html to enable two players play through internet."*

## Architecture

Each game is a single HTML file with inline CSS and JavaScript. All game state lives in plain JS variables. Canvas 2D API is used for the Go board; the Tic Tac Toe board uses CSS Grid.

**go.html** uses one external dependency (PeerJS CDN) for multiplayer networking. Local play requires no server.

### go.html internals
- `board` — `Int8Array[19][19]`: 0=empty, 1=black, 2=white
- `placeStone(r, c)` — validates move (suicide, ko via `boardsEqual`), captures opponent groups, updates state
- `getGroup(r, c, b)` — flood-fill returning all stones in a connected group
- `computeScore()` — flood-fills empty regions; assigns territory to whichever single color borders each region; counts dead stones per color
- Scoring phase triggered by two consecutive passes; dead stone groups toggled by click via `deadSet` (keyed as `r * SIZE + c`)

### go.html multiplayer internals
- Uses PeerJS (WebRTC) for peer-to-peer connection via free hosted signaling server
- Host creates room with 6-char code, guest joins with code or URL `?room=CODE`
- Host-authoritative model: host validates all moves, broadcasts state to guest
- `myColor` — 0=local game, 1=black, 2=white (online)
- `isHost` — true if this player created the room
- `sendState()` — serializes full game state and sends to peer
- `applyState()` — deserializes and applies received state
- Message types: `move`, `pass`, `resign`, `state`, `toggleDead`, `countScore`, `resumePlay`, `chat`, `newGame`
- Reconnection: guest auto-retries every 3 seconds if disconnected
- Connection options: `{ serialization: 'json', reliable: true }` required for TURN relay compatibility

### go.html hosting & access
- Hosted on GitHub Pages: https://hytsang.github.io/ClaudeProject/go.html
- Password protected with SHA-256 hashed access code (stored in `ACCESS_HASH`)
- TURN servers provided by Metered.ca (free tier, 500GB/month) for NAT traversal
- TURN credentials stored in `PEER_CONFIG.config.iceServers`

### Updating TURN credentials
If TURN stops working, get new credentials from metered.ca/stun-turn:
1. Sign up for free account
2. Go to Dashboard → TURN → Free TURN Server Credentials
3. Update the `username` and `credential` fields in `PEER_CONFIG`
