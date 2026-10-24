# CLAUDE.md

Guidance for Claude Code when working in this repository.

## What this repo is

A single-file browser game, **Sky Fighter** (a Raiden-style vertical shoot-'em-up).
There is no build step, no package manager, and no test suite.

- `index.html` — the entire game: inline `<style>`, markup, and one inline `<script>` IIFE (~1,670 lines).
- `README.md` — minimal project title only.

## Running it

Open `index.html` directly in a browser, or serve the folder with any static server
(e.g. `python3 -m http.server`). Nothing needs to be installed.

## Verifying changes

There are no automated tests or linters. For any gameplay or UI change:

1. Load the page in a browser and check the browser console for errors.
2. Play through at least one stage, including a boss fight, and confirm the change behaves as intended.
3. Check the touch layout (joystick, fire, bomb buttons) at phone width if the change touches layout or input.
4. Check the pause (`P` / `Esc`) and game-over flows if the change touches state transitions.

If you cannot run a browser, say so explicitly and verify by reading the code.

## Code layout inside `index.html`

The script is one IIFE with `'use strict'`. Everything lives inside it, so nothing is global.

| Area | Key names |
| --- | --- |
| Config tables | `DIFFICULTIES`, `THEMES`, `ENEMY_TYPES`, `BOSSES`, `BOSS_PATTERNS`, `KILLS_PER_STAGE` |
| Persistence | `STORAGE_*` constants (`localStorage` keys prefixed `skyfighter.`) |
| Audio | `ensureAudio`, `beep`, `noiseBurst`, `sfx` (Web Audio, synthesized — no audio files) |
| Input | keyboard listeners, mouse on canvas, touch joystick and buttons |
| Game state | `resetGame()` initializes all mutable state |
| Spawning | `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp` |
| Simulation | `update(dt)`, `updateBoss(dt)`, `applyPowerUp`, `useBomb`, `fireWeapon` |
| Rendering | `draw()`, `drawPlane`, `drawBossShip`, `drawClouds` (all canvas 2D, no sprites) |
| Loop | `loop(now)` drives `update` and `draw` via `requestAnimationFrame`; `dt` is clamped to 50 ms |

### Gameplay facts worth knowing

- Three difficulties (`easy` / `normal` / `hard`) set spawn rates, enemy shoot chance, lives, bombs, and boss HP multiplier.
- A stage ends after `KILLS_PER_STAGE` (18) kills, which triggers a boss. Stage N uses `THEMES[(N-1) % THEMES.length]` and `BOSSES[(N-1) % BOSSES.length]`, so both cycle.
- Enemies: `grunt`, `interceptor` (weaves), `tank` (4 HP, appears from stage 2).
- Power-ups: `life`, `bomb`, `weapon` (raises weapon level to 3, then toggles between `vulcan` and `laser`).
- An adaptive quality setting (`quality` 0–2) lowers visual detail when frame times are slow.
- Controls: arrows or WASD to move, Space or hold-to-fire to shoot, `X` or Shift to bomb, `P` or `Esc` to pause.
- Reduced motion is honored from `prefers-reduced-motion` and can be toggled in the UI.

## Conventions

- Match the existing style: 2-space indentation, single quotes, `const`/`let`, arrow functions for callbacks, short section comments with `// ---------- Name ----------`.
- Keep everything in the existing single file unless the user asks for a split. Do not add a build step or dependencies without a request.
- Keep the game's data-driven tables (`ENEMY_TYPES`, `BOSSES`, `DIFFICULTIES`) as the place to tune numbers rather than hard-coding them into logic.
- Any new `localStorage` key must use the `skyfighter.` prefix and be declared alongside the existing `STORAGE_*` constants.
- Preserve accessibility hooks: the `srLive` live region is used for screen-reader announcements via `announce()`.

## Git

- Develop on the branch assigned for the session; do not push to other branches without permission.
- Commit messages should be short and descriptive of the change.
