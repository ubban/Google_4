# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo contains **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs in the browser. There is no build step, no package manager, and no test suite. The whole game is one self-contained file:

- `index.html` (~1,670 lines): markup, CSS (`<style>`, roughly lines 10–361), canvas element, and all game logic in one inline `<script>` IIFE (starting ~line 422).
- `README.md`: a placeholder. It has no setup instructions.

Open `index.html` directly in a browser to play. No server is required.

## Repository layout and code map

Everything lives inside a single `(function () { 'use strict'; ... })()` closure, so nothing leaks onto `window`. Key sections of the script, in file order:

| Area | Approx. location | Notes |
| --- | --- | --- |
| DOM refs | ~425–460 | `getElementById` lookups for HUD, overlays, touch controls, and buttons |
| Storage keys | ~463–466 | `skyfighter.bestScore`, `.muted`, `.difficulty`, `.reducedMotion` |
| Tuning tables | ~485–527 | `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES`, `ENEMY_TYPES`, `BOSSES` |
| Audio | ~545–680 | Web Audio synth: `ensureAudio`, `beep`, `noiseBurst`, `EXPLODE_SFX` |
| HUD / FX helpers | ~680–750 | hearts, bombs, combo pop, stage banner, screen shake, difficulty switch |
| Layout and input | ~753–870 | `resize` (fixed 480×720 base), virtual joystick, mouse/touch/keyboard handling |
| Entities and spawning | ~875–1150 | `resetGame`, `drawPlane`, `drawBossShip`, `spawnEnemy`, `spawnBoss`, power-ups, bombs, `fireWeapon` |
| Update loop | ~1148–1303 | `update(dt)`: movement, collisions, scoring, stage progression |
| Power-ups and boss patterns | ~1304–1410 | `applyPowerUp`, `BOSS_PATTERNS` (radial, spread, spiral, cross, homing, double-ring) |
| Boss and game state | ~1411–1506 | `updateBoss`, `killBoss`, `loseLife`, `endGame`, `togglePause` |
| Render | ~1507–1639 | `drawClouds`, `draw()` |
| Main loop | ~1640–end | `loop(now)` drives `requestAnimationFrame` |

Line numbers are approximate and will drift as the file changes. Prefer searching by function name.

## Conventions

- **Keep it a single file** unless the user asks for a split. Do not introduce a bundler, framework, or npm dependencies for this game.
- **Keep the IIFE and `'use strict'`.** New helpers belong inside the closure, not on `window`.
- **Tune gameplay through the existing tables** (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `BOSS_PATTERNS`, `THEMES`) rather than hard-coding magic numbers in update or draw code.
- **Time-based updates use `dt`.** Movement and timers in `update(dt)` should scale with `dt` so behaviour stays consistent across frame rates.
- **Persist settings only through the `skyfighter.*` localStorage keys.** Wrap reads and writes so the game still works if storage is unavailable (private mode, blocked site data).
- **Respect reduced motion.** Screen shake, flashes, and similar effects should check the `reducedMotion` setting, which is also persisted.
- **Accessibility.** Announcements go through `announce()` and the `srLive` live region. Keep overlays and buttons keyboard-reachable.
- **Input is three-way.** Keyboard (`keydown`/`keyup` on `e.code`), mouse/pointer via `canvasPointFromEvent`, and touch (joystick plus fire/bomb buttons). Changes to controls should be checked on all three where relevant.
- **Canvas is scaled, not resized per frame.** Drawing uses the 480×720 base coordinate system; `resize()` handles fit-to-viewport.
- **Match the existing style**: 2-space indentation, single quotes in JS, `const` by default, small named functions, and comments only where the logic is non-obvious.

## Working on the game

- To **run it**, open `index.html` in a browser, or serve the directory with any static server (for example `python3 -m http.server`) if a feature needs `http://` origin.
- There is **no automated test suite and no linter** configured. Verify changes by playing them in a browser. For UI or gameplay changes, check at least:
  - Start, pause, resume, game over, and restart.
  - Each difficulty level (`easy`, `normal`, `hard`).
  - Stage progression and boss fights (boss HP bar and attack patterns).
  - Bomb, power-up, and weapon-level behaviour.
  - Mute and reduced-motion toggles persisting across reloads.
  - Touch layout at phone width, if the change touches the HUD or controls.
- If you cannot run a browser in the environment, say so explicitly and limit claims to what was verified by reading the code.
- Syntax check the inline script after edits if practical, for example by extracting the `<script>` block and running `node --check` on it.

## Git and delivery

- Default branch is the one checked out for the session. Develop on the branch named in the session instructions, and commit and push only when asked or when the session instructions say to.
- Keep commits focused. Describe gameplay changes in the commit message (what changed and why), not just the file touched.
- Do not open a pull request unless the user asks for one.

## Things to avoid

- Do not add a build pipeline, bundler, or framework without an explicit request.
- Do not move the game logic out of the IIFE into globals.
- Do not change the `skyfighter.*` storage key names; they would reset players' saved best scores and settings.
- Do not rewrite large parts of `index.html` in one edit when a targeted change will do. The file is large and most sections are interdependent through shared state inside the closure.
