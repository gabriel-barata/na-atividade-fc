# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A single-file, static web app: a pickup-soccer ("pelada") team lineup builder for "Na Atividade Futebol Clube". Everything — HTML, CSS, and JS — lives in `index.html`. There is no build step, no package manager, no framework, and no test suite.

## Running / developing

Open `index.html` directly in a browser, or serve it locally, e.g.:

```
python3 -m http.server
```

There is no build, lint, or test command — changes are made directly to `index.html` and verified by reloading in a browser.

## Architecture

The whole app is an IIFE in the `<script>` block at the bottom of `index.html`, organized around a single global `state` object persisted to `localStorage` (`pelada:escalacao:v1`) via `save()`/`load()`.

- **State shape**: `{teamCount, nextId, players[], teamNames:{A,B,C}, nextMatch}`. Each player is `{id, name, skill (1-100), pos (GOL/LAT/ZAG/VOL/MEI/ATA), team ('pool'|'A'|'B'|'C'), num, fx, fy}` — `fx`/`fy` are fractional coordinates for the player's dot position on the mini pitch.
- **Roster source**: `data/players.json` is the canonical list of players (`name`, `num`, `pos`, `skill`) used to seed a fresh `state` the first time the app runs on a device (`load()` fetches it when `localStorage` has no saved state yet). After that, `localStorage` is the source of truth — editing `data/players.json` only affects new/cleared browser profiles, not devices with existing saved state. `fetch()` requires the page to be served over http(s) (works on GitHub Pages and `python3 -m http.server`); opening `index.html` directly via `file://` will fail the fetch and fall back to an empty roster.
- **Rendering**: fully re-render on every mutation via `render()`, which rebuilds the pool column and team columns (`teamsEl.innerHTML = ...`) from `state`. There's no virtual DOM/diffing — direct string-concatenated HTML templates (`playerHTML`, `pitchHTML`, etc.).
- **Event handling**: all delegated on `document` (`click`, `input`, `change`, `pointerdown`, drag events) using `data-*` attributes to identify targets (`data-move`, `data-del`, `data-name`, `data-skill`, `data-pos`, `data-nick`, `data-drop`, etc.), rather than per-element listeners attached during render.
- **Two drag interactions coexist**: native HTML5 drag-and-drop (desktop) for moving player cards between the pool/team columns, and pointer-event dragging for repositioning a player's dot within a team's mini pitch (`fx`/`fy`). Mobile uses the A/B/C/D `.mv` buttons in `movebar` instead of dragging cards.
- **Team assignment logic**: `autoBalance()` seeds one goalkeeper (GOL) per team first, then greedily assigns remaining players sorted by skill descending to whichever team currently has the fewest players (tie-broken by lowest total skill).
- **Persistence/portability**: state autosaves to `localStorage` on every mutation; `exportJSON()`/`importJSON()` let users download/upload the full state as JSON for backup or sharing between devices (data is explicitly local-only, no server/backend).
- **2 vs 3 teams**: `state.teamCount` toggles between team sets `['A','B']` and `['A','B','C']`; switching to 2 teams sends any players on team C back to the pool.

## Conventions

- UI copy is in Portuguese (pt-BR); keep new user-facing strings consistent with that.
- Vanilla ES5-style JS (`var`, function expressions, no modules/classes) — match this style rather than introducing ES6+ syntax or a bundler.
- CSS custom properties (`--p900`...`--p100` purple scale) define the color system in `:root`; reuse these rather than hardcoding new colors.
