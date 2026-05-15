# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

A collection of browser-playable games implemented as self-contained single HTML files (no build step, no dependencies, no server required). Open any file directly in a browser with `open <file>.html`.

## Files

- `tictactoe.html` — 2-player Tic Tac Toe with score tracking
  - Generated with prompt: *"make tic tac toe that i can play on the web"*
- `go.html` — 19×19 Go with full Japanese rules scoring (territory counting, dead stone marking, komi)
  - Generated with prompt: *"make a 19 by 19 Go game that i can play on web. make it auto count the result with Japanese Go rules."*

## Architecture

Each game is a single HTML file with inline CSS and JavaScript — no external dependencies, no modules, no bundler. All game state lives in plain JS variables. Canvas 2D API is used for the Go board; the Tic Tac Toe board uses CSS Grid.

### go.html internals
- `board` — `Int8Array[19][19]`: 0=empty, 1=black, 2=white
- `placeStone(r, c)` — validates move (suicide, ko via `boardsEqual`), captures opponent groups, updates state
- `getGroup(r, c, b)` — flood-fill returning all stones in a connected group
- `computeScore()` — flood-fills empty regions; assigns territory to whichever single color borders each region; counts dead stones per color
- Scoring phase triggered by two consecutive passes; dead stone groups toggled by click via `deadSet` (keyed as `r * SIZE + c`)
