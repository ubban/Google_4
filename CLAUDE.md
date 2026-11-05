# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo contains **Sky Fighter**, a single-page HTML5 canvas vertical shoot-'em-up (Raiden-style). There is no build step, package manager, or test suite.

- `index.html` — the entire game: markup, CSS (`<style>`), and JavaScript (`<script>`, wrapped in one IIFE with `'use strict'`).
- `README.md` — minimal project readme.

To run the game, open `index.html` in a browser. No server is required.

## Code layout (`index.html`)

Approximate regions, in file order:

1. **Head / CSS** (lines ~1–362): dark color scheme, HUD pills, glassmorphic overlay card, touch controls, and animation keyframes.
2. **Markup** (lines ~363–421): `#gameCanvas`, HUD elements (score, best, multiplier, weapon, hearts, bombs, boss bar), overlay, touch controls, and a `#srLive` aria-live region.
3. **Script** (lines ~422 onward):
   - DOM references and `STORAGE_*` keys (`skyfighter.*` in `localStorage`).
   - Config tables: `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES`, `ENEMY_TYPES`, `BOSSES`, `BOSS_PATTERNS`.
   - Audio: a Web Audio synth (`beep`, `noiseBurst`); audio is created lazily via `ensureAudio()` on first user gesture.
   - Input: keyboard (`keys` map), mouse on canvas, and touch joystick/fire/bomb buttons.
   - Game state and logic: `resetGame`, `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`, `fireWeapon`, `useBomb`, `update(dt)`, `updateBoss`, `loseLife`, `endGame`.
   - Rendering: `draw()`, `drawPlane`, `drawBossShip`, `drawClouds`.
   - Main loop: `loop(now)` via `requestAnimationFrame`.

## Game rules worth knowing

- **Difficulty** (`DIFFICULTIES`): `easy`, `normal` (default), `hard`. Each sets spawn rate, enemy speed, enemy fire chance, starting lives, starting bombs, and boss HP multiplier.
- **Stages**: a stage advances every `KILLS_PER_STAGE` (18) kills and ends in a boss. Bosses cycle through `BOSSES` by `(stage - 1) % BOSSES.length`, and each boss uses the matching entry in `BOSS_PATTERNS`.
- **Weapons**: Vulcan and Laser, each with power levels; power-ups include weapon level, bomb, and extra life.
- **Scoring**: a kill-chain multiplier (`addScore`, `breakChain`).

## Controls

| Action | Keyboard | Touch |
| --- | --- | --- |
| Move | Arrow keys or WASD | Left-side joystick |
| Fire | Space (hold) | Fire button (hold) |
| Bomb | X or Shift | Bomb button |
| Pause | P or Escape | Pause button |
| Aim / move | — | Mouse position on canvas (`mouseActive`) |

## Persistence

Settings and best score are stored in `localStorage` under these keys. Always wrap access in try/catch if you add new storage, since storage can be blocked (private mode, disabled site data).

- `skyfighter.bestScore`
- `skyfighter.muted`
- `skyfighter.difficulty`
- `skyfighter.reducedMotion`

Existing code reads these directly; if you change a key name, migrate existing saved values.

## Conventions

- Keep everything in `index.html` unless there is a clear reason to split files. The single-file, no-build design is intentional.
- Stay in vanilla JS and the 2D canvas API. Do not add frameworks or a bundler without asking.
- Preserve the `'use strict'` IIFE structure and avoid leaking globals.
- Game-logic tuning lives in the config tables at the top of the script (`DIFFICULTIES`, `BOSSES`, `BOSS_PATTERNS`, etc.). Prefer changing those over hard-coding numbers in the update loop.
- Support both desktop and touch input when adding controls, and keep the layout usable at phone width.
- Respect the reduced-motion setting when adding screen effects (shake, flashes, pulses).
- Announce important state changes through `announce()` so screen readers get them.

## Testing and verification

There are no automated tests. To verify a change:

1. Open `index.html` in a browser and play through a full stage to a boss.
2. Check each difficulty, pause/resume, mute, and reduced-motion toggles.
3. Check touch controls with device emulation at phone width.
4. Confirm the best score persists across reloads.

For visual changes, take a screenshot before and after when practical.

## Git

- Work on the feature branch assigned for the session; do not push to other branches without permission.
- Commit messages should describe the gameplay or code change in plain terms.
