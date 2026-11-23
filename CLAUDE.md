# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo is a single-page browser game, **Sky Fighter** — a Raiden-style vertical shoot-'em-up.

- `index.html` — the entire game: markup, CSS (`<style>`, ~lines 10–362), canvas element, and one inline `<script>` (~lines 422–1667) that wraps all logic in an IIFE with `'use strict'`.
- `README.md` — minimal placeholder; no build or setup instructions.
- There is no package manager, bundler, test suite, or CI configuration. Nothing needs installing.

## Running and testing

- Open `index.html` directly in a browser, or serve the directory with any static server (e.g. `python3 -m http.server`).
- There are no automated tests or linters. Verify changes by playing the game in a browser. For UI or gameplay changes, load the page and exercise the affected path (start, fire, bomb, pause, boss fights, game over).

## Code structure (inside the script)

All state and functions live in the single IIFE. Key areas, roughly in file order:

| Area | What it holds |
|---|---|
| DOM refs | `const` handles to every element by id (`scoreEl`, `startBtn`, `overlay`, …) |
| Storage keys | `STORAGE_BEST`, `STORAGE_MUTE`, `STORAGE_DIFF`, `STORAGE_REDUCED` (all `skyfighter.*` in `localStorage`) |
| Config tables | `DIFFICULTIES` (easy/normal/hard tuning), `THEMES`, `ENEMY_TYPES`, `BOSSES`, `BOSS_PATTERNS` |
| Audio | `ensureAudio`, `beep`, `noiseBurst`, `EXPLODE_SFX` — Web Audio synthesis, no audio files |
| HUD / feedback | `renderHearts`, `renderBombs`, `showComboPop`, `showStageBanner`, `screenShake`, `flashHit` |
| Layout / input | `resize` (fixed `BASE_W=480`, `BASE_H=720` logical canvas), joystick and touch handlers, keyboard handlers |
| Game lifecycle | `resetGame`, `update(dt)`, `draw()`, `loseLife`, `endGame`, `togglePause`, `loop(now)` |
| Entities | `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`, `spawnExplosion`, `applyPowerUp`, `fire*` pattern helpers |
| Perf | `trackPerf` adjusts a `quality` level (0–2) based on average frame time |

Gameplay constants worth knowing:
- `KILLS_PER_STAGE = 18` — kills needed to advance a stage. The visual theme is picked as `THEMES[(stage - 1) % THEMES.length]`.
- Bosses are drawn in `drawBossShip` and driven by `updateBoss`; `killBoss` handles defeat.
- Controls: arrows / WASD to move, Space or hold-to-fire (touch: fire button), X / Shift for bomb, P / Escape to pause. Touch devices get an on-screen joystick and fire/bomb buttons.

## Conventions

- Keep everything in `index.html` unless there is a clear reason to split files. Do not introduce a build step, framework, or external dependency without asking.
- Match the existing style: 2-space indentation, `const`/`let`, arrow functions for callbacks, `camelCase` for functions and variables, `UPPER_SNAKE_CASE` for config constants.
- Tune gameplay by editing the config tables (`DIFFICULTIES`, `BOSSES`, `ENEMY_TYPES`) rather than hard-coding numbers in logic.
- Persist user settings only through the existing `skyfighter.*` `localStorage` keys, and wrap new storage reads/writes in the same style already used.
- Respect accessibility hooks that already exist: the `reducedMotion` flag (seeded from `prefers-reduced-motion`, toggled by the motion button) dampens `screenShake` and skips the flash effects. `srLive` is the screen-reader announcement region, written via `announce()`. New visual effects should honor `reducedMotion`.
- Keep the game playable at phone width; the layout scales the canvas through `resize()`.

## Git workflow

- Develop on the branch assigned to the session (currently `claude/gifted-tesla-9vkr5p`). Do not push to other branches without explicit permission.
- Do not open pull requests unless asked.
