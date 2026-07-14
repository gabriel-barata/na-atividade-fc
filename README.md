# Na Atividade Futebol Clube

A simple web app for organizing our pickup soccer ("pelada") games — add players, set their skill and position, and split them into balanced teams. Live on GitHub Pages.

## Features

- Add/remove players and edit name, jersey number, position, and skill (1–100) inline
- Split the group into 2 or 3 teams, dragging players between the pool and teams (desktop) or using the D/A/B/C buttons (mobile)
- **Auto-nivelar**: automatically balances teams by skill, one goalkeeper per team
- Balance indicator showing how even the teams' average skill is
- Drag player dots to position them on each team's mini pitch
- Copy the lineup as text, or export/import the full state as a JSON backup
- Set the date of the next match

## Running locally

This is a static site with no build step — just serve the folder and open it:

```
python3 -m http.server
```

Then visit `http://localhost:8000`. Opening `index.html` directly by double-clicking it (`file://`) won't work, since the app fetches `data/players.json` and browsers block that over `file://`.

## Project structure

- `index.html` — the entire app (HTML, CSS, and JS in one file)
- `data/players.json` — the default player roster (name, jersey number, position, skill), used to seed a new browser's saved state
- `assets/logo.png` — club crest

## Editing `data/players.json`

This file is the default roster the app loads for anyone opening it fresh (see [Data & persistence](#data--persistence) below for what "fresh" means). It's a JSON array, one object per player:

```json
{"name": "Garcia", "num": 1, "pos": "MEI", "skill": 84}
```

| Field   | Type   | Notes |
|---------|--------|-------|
| `name`  | string | Display name. |
| `num`   | number | Jersey number. Shown on the player card and on the mini pitch dot. Doesn't need to be unique, but should be to tell players apart at a glance. |
| `pos`   | string | Position code — one of `GOL`, `LAT`, `ZAG`, `VOL`, `MEI`, `ATA` (see table below). Sets the player's default spot on the mini pitch and is used by **Auto-nivelar** to place one goalkeeper (`GOL`) per team. |
| `skill` | number | Skill rating, `1`–`100`. Drives both the color of the skill badge and the team-average balance calculation. |

Position codes (must be entered exactly as shown, uppercase):

| Code  | Meaning (pt-BR)  |
|-------|------------------|
| `GOL` | Goleiro          |
| `LAT` | Lateral          |
| `ZAG` | Zagueiro         |
| `VOL` | Volante          |
| `MEI` | Meio-campo       |
| `ATA` | Atacante         |

To add a player, add a new `{...}` object to the array (comma-separated, no trailing comma after the last one). To remove one, delete their object. Validate the file is well-formed JSON before committing, e.g.:

```
python3 -m json.tool data/players.json > /dev/null && echo ok
```

## Data & persistence

Everything a player edits in the app itself (skill, position, team assignments, etc.) is saved to that browser's `localStorage`, not written back to `data/players.json`. Editing `data/players.json` only affects fresh browser profiles that haven't saved state yet — existing players/devices won't see the update automatically. Use **Exportar**/**Importar** in the app to back up or share a specific lineup as a JSON file.

See `CLAUDE.md` for architecture notes if you're changing the code.
