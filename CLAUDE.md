# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo contains **Sky Fighter**, a Raiden-style vertical shoot-em-up that runs in the browser on an HTML `<canvas>`. The entire game is one file: `index.html`, with inline `<style>` and an inline `<script>`. `README.md` is a placeholder.

There is no build step, package manager, bundler, framework, or test suite.

## Running the game

- Open `index.html` directly in a browser, or serve the directory with any static server (for example `python3 -m http.server`).
- There is nothing to install or compile.

## Code layout (`index.html`)

- **`<head>`/`<style>` (about lines 1–360):** CSS for the HUD, overlays, touch controls, and themes. Keep styles in this block; do not add a stylesheet file.
- **`<body>` (about lines 361–421):** static DOM. The canvas is `#gameCanvas`. Overlays, HUD elements, buttons (`#startBtn`, `#pauseBtn`, `#muteBtn`, `#motionBtn`, `.diffBtn`), touch controls (`#touchControls`, `#moveZone`, `#fireBtn`, `#bombBtn`), and the `#srLive` live region are all declared here. JavaScript looks them up by ID, so renaming an ID means updating the script too.
- **`<script>` (about lines 422–1669):** one IIFE with `'use strict'`. It is organized into these sections:
  - **Config tables:** `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES`, `ENEMY_TYPES`, `BOSSES`, and `BOSS_PATTERNS` (bullet-pattern functions).
  - **Persistence:** `localStorage` keys prefixed `skyfighter.` (`bestScore`, `muted`, `difficulty`, `reducedMotion`). Every access is a plain call with no try/catch, so keep that in mind if you change storage handling.
  - **Audio:** a WebAudio context (`ensureAudio`) and a synthesized `sfx` set built from `beep` and `noiseBurst`. There are no audio asset files.
  - **Input:** keyboard handlers, touch joystick, fire and bomb buttons, and mouse drag on the canvas.
  - **Game state:** module-level `let` variables (`score`, `lives`, `stage`, `kills`, `chain`, `multiplier`, `bombCount`, and so on). `resetGame()` reinitializes them.
  - **Spawning and entities:** `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`, `spawnExplosion`, and the weapon functions (`fireWeapon` and `fire*` helpers).
  - **Update and draw:** `update(dt)`, `updateBoss(dt)`, and `draw()`. Drawing is done with `drawPlane`, `drawBossShip`, and `drawClouds`.
  - **Main loop:** `loop(now)` calls `trackPerf`, then `update` and `draw` while the game is running and not paused. `dt` is clamped to 50 ms.

## Game mechanics to keep in mind

- **Stages:** each stage is `KILLS_PER_STAGE` (18) kills. After that, a boss spawns about 1.2 s later. Stage themes come from `THEMES`, indexed by stage.
- **Difficulty:** `easy`, `normal`, and `hard` change spawn timing, enemy speed, enemy fire chance, starting lives, starting bombs, and boss HP multiplier.
- **Enemies:** `grunt`, `interceptor` (weaving), and `tank` (4 HP, appears from stage 2).
- **Adaptive quality:** `trackPerf` watches the average frame time over 45 frames and steps a `quality` level from 0 to 2 to reduce effects when the frame rate drops.
- **Controls:** WASD or arrow keys to move, Space to fire, X or Shift for a bomb, P or Escape to pause. On touch devices, a virtual joystick and buttons are shown.

## Conventions

- Keep the game as a single self-contained `index.html` unless the user asks for a restructure. Do not introduce a build system or external dependencies without asking.
- Match the existing style: 2-space indentation, single quotes in JS, `const` by default, and `let` for mutable game state.
- Game tuning values live in the config tables near the top of the script. Prefer changing those over hardcoding numbers in the update logic.
- Accessibility features are part of the design. Preserve the `#srLive` announcements, reduced-motion handling (`reducedMotion`), and the keyboard and touch controls when you change input or UI code.
- Use the `announce()` helper for screen-reader messages instead of writing to `#srLive` directly.

## Verification

There are no automated tests or linters. To verify a change:

1. Open `index.html` in a browser, or serve it locally.
2. Check the browser console for errors.
3. Play through at least one start, fire, bomb, pause, and game-over cycle. If the change touches stages or bosses, reach the boss.
4. On touch layouts, check the controls at phone width, or use a mobile emulation mode.

If you can run a headless browser (Playwright with the preinstalled Chromium is available in this environment), it is a good way to smoke-test the page. Use it for checks only; do not add test tooling to the repo without asking.

## Git

- Develop on the branch assigned for the session. Do not push to other branches without permission.
- Do not open pull requests unless the user asks.
