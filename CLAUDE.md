# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo contains **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs in the browser. It is a single self-contained file:

- `index.html` (~1,670 lines): all markup, CSS (`<style>`, starts ~line 10), and game logic (one inline `<script>`, starts ~line 422, wrapped in an IIFE with `'use strict'`).
- `README.md`: a stub. Keep it minimal unless the user asks for docs.

There is no build step, bundler, package manager, test suite, or linter. Nothing needs to be installed.

## Running and verifying

- Open `index.html` directly in a browser, or serve the directory with any static server (e.g. `python3 -m http.server`).
- Game rendering uses a `<canvas id="gameCanvas">` with a 2D context. Logical size is `BASE_W = 480`, `BASE_H = 720`, scaled by `resize()`.
- There are no automated tests. For gameplay changes, verify in a real browser (desktop and a touch/narrow viewport) and check for console errors.

## Code map (inside `index.html`'s script)

- **Config tables**: `DIFFICULTIES` (easy/normal/hard: spawn rates, enemy speed, shoot chance, lives, bombs, boss HP multiplier), `KILLS_PER_STAGE` (18), `THEMES` (per-stage sky/cloud/sun palettes; cycled by stage), `ENEMY_TYPES`, `BOSSES`, `BOSS_PATTERNS`.
- **Persistence**: `localStorage` keys are prefixed `skyfighter.` (`bestScore`, `muted`, `difficulty`, `reducedMotion`). Keep the prefix and read/write them defensively, since storage can be unavailable.
- **Audio**: Web Audio via `ensureAudio()`, `beep()`, `noiseBurst()`. Audio must start from a user gesture (`ensureAudio()` is called in the start button handler).
- **Input**: keyboard (`window` `keydown`/`keyup`: arrows/WASD move, Space fires, X or Shift bombs, P/Esc pauses), mouse and pointer steering on the canvas (`canvasPointFromEvent`), and touch (`joystickStart/Move/End`, `FIRE` and `BOMB` buttons).
- **Simulation**: `resetGame()` initializes state, `update(dt)` advances it, `spawnEnemy`, `spawnBoss`, `fireWeapon`, `useBomb`, `applyPowerUp`, `updateBoss`, `loseLife`, `endGame`, `togglePause`.
- **Rendering**: `draw()` renders the frame. Helpers: `drawPlane`, `drawBossShip`, `drawClouds`.
- **Loop**: `loop(now)` runs on `requestAnimationFrame`, clamps `dt` to 0.05 s, and skips update/draw when not running or paused.
- **Accessibility and UI**: `announce()` writes to the `srLive` live region. Reduced-motion toggle (`motionBtn`) controls screen shake and flashes. HUD elements are updated via `renderHearts`, `renderBombs`, `showComboPop`, `showStageBanner`.

## Conventions

- Keep everything in `index.html` unless the user asks for a split. Don't add a build system or dependencies without asking.
- Match the existing style: 2-space indentation, single quotes in JS, `const`/`let`, small named functions, and comments only where the logic is non-obvious.
- Put tunable numbers in the config tables (`DIFFICULTIES`, `BOSSES`, `ENEMY_TYPES`, etc.) rather than hard-coding them in logic.
- Game time must go through `dt` (seconds) so behavior is frame-rate independent.
- Preserve the mobile layout: the touch controls and HUD must stay usable at phone widths.
- Preserve reduced-motion behavior when adding new screen shake, flash, or pulse effects.
- Keep accessibility hooks (`aria-live` announcements, `title`/labels on icon buttons) working when changing UI.

## Git

- Develop on the branch designated for the session (currently `claude/gifted-tesla-n2dj1l`). Don't push to other branches without permission.
- Commit only when asked. Don't open pull requests unless the user asks for one.
