# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo is a small, dependency-free web game: **Sky Fighter**, a Raiden-style
vertical shoot-em-up. The whole game is one self-contained file, `index.html`
(inline CSS, one inline `<script>`, HTML5 `<canvas>`). `README.md` is a stub.

There is no build step, no package manager, no test suite, and no linter
configured.

## Repository layout

| Path        | Purpose                                                        |
|-------------|----------------------------------------------------------------|
| `index.html`| The entire game: markup (HUD, overlay, touch controls), CSS, and JS |
| `README.md` | Minimal project title                                          |
| `CLAUDE.md` | This file                                                      |

Inside `index.html` the script is one IIFE (`(function () { 'use strict'; ... })();`)
and is organized by section comments (`// ---------- Input ----------`, etc.).
Key areas, in file order:

- **Config tables**: `DIFFICULTIES`, `THEMES` (stage backgrounds), `ENEMY_TYPES`,
  `BOSSES`, `BOSS_PATTERNS`, `KILLS_PER_STAGE`.
- **Audio**: `ensureAudio`, `beep`, `noiseBurst`. Synthesized with WebAudio; no audio files.
- **HUD helpers**: `renderHearts`, `renderBombs`, `showComboPop`, `showStageBanner`, `announce`.
- **Input**: keyboard `keys` map, mouse/touch joystick, fire and bomb buttons.
- **Game state**: `resetGame`, `update(dt)`, `spawnEnemy`, `spawnBoss`, `applyPowerUp`,
  `useBomb`, `fireWeapon`, `loseLife`, `endGame`, `togglePause`.
- **Rendering**: `draw`, `drawPlane`, `drawBossShip`, `drawClouds`.
- **Loop**: `loop(now)` drives `update` then `draw` each frame, with `dt` clamped to 0.05 s.
  `trackPerf` adjusts a `quality` level (0/1/2) based on frame times.

## Running the game

No install needed. Open the file directly:

```bash
xdg-open index.html        # Linux
open index.html            # macOS
```

Or serve it statically (useful if a browser restricts `file://` features):

```bash
python3 -m http.server 8000   # then visit http://localhost:8000/
```

Requires a modern browser with Canvas 2D, WebAudio, and `localStorage`.

## Controls (as implemented)

- Move: mouse, Arrow keys, or WASD. Touch: drag on the left zone.
- Fire: Space (hold), or click/tap the FIRE button.
- Bomb: `X` or either Shift (clears the screen, damages bosses). Touch: BOMB button.
- Pause: `P` or `Esc`.

## Persistence

Settings and scores are stored in `localStorage` under these keys. Keep the names
stable, because changing them resets players' data:

- `skyfighter.bestScore`
- `skyfighter.muted`
- `skyfighter.difficulty`
- `skyfighter.reducedMotion`

Reads must handle a missing key: best score uses `Number(...) || 0`, and the flags compare
the raw string (for example `=== '1'`). Preserve that fallback behavior.

## Conventions

- **Single file.** Keep the game in `index.html` unless there is a clear reason to split it.
  Do not add a bundler, framework, or npm dependency without asking.
- **Vanilla JS, ES2015+.** Use `const`/`let`, arrow functions, and template strings as the
  existing code does. Stay inside the IIFE; do not add globals.
- **Game tuning lives in data tables.** Prefer changing values in `DIFFICULTIES`,
  `ENEMY_TYPES`, `BOSSES`, or `THEMES` over hard-coding numbers in logic.
- **Frame-rate independence.** Movement and timers use `dt` (seconds). Keep new motion
  in that form.
- **Accessibility.** Respect `reducedMotion` for screen shake and flashes, and route
  player-facing status text through `announce()` for screen readers.
- **Mobile.** Canvas scales to fit via `resize()` (base size 480×720). Test changes to
  HUD or touch layout at phone width.

## Verifying changes

There are no automated tests. Before committing gameplay or rendering changes:

1. Open the game in a browser and start a run on each difficulty.
2. Check the touched path: a new enemy or boss pattern should spawn and die correctly,
   power-ups should apply, and pause/restart should work.
3. Check the browser console for errors. A clean console is the minimum bar.
4. For layout or HUD changes, check both a desktop and a phone-sized viewport.

If you add a test harness later, document the command here.

## Git

- Development branch for this work: `claude/gifted-tesla-5v3aot`.
- Write clear commit messages that describe the gameplay or code change.
- Do not open pull requests unless asked.
