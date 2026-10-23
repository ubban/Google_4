# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Google_4 is a small, dependency-free browser game, **Sky Fighter**: a Raiden-style vertical shoot-'em-up. The entire game lives in a single file, `index.html` (inline CSS and one inline `<script>` IIFE, about 1,670 lines). `README.md` is a placeholder.

There is no build step, package manager, test suite, linter, or CI configuration. Do not add tooling unless the user asks for it.

## Running the game

Open `index.html` directly in a browser, or serve the directory with any static server (for example `python3 -m http.server`). Nothing needs to be installed.

## Code layout (`index.html`)

- **`<style>`**: all styling, including the HUD, overlay, and touch controls.
- **`<body>`**: the `<canvas id="gameCanvas">`, the overlay (title, stats, start button), HUD elements, and touch controls (`#touchControls`, `#moveZone`, `#fireBtn`, `#bombBtn`).
- **Single `<script>`**: wrapped in `(function () { 'use strict'; ... })()`. Sections are marked with `// ---------- Name ----------` comments:
  - **Adaptive performance**: `trackPerf` lowers or raises the `quality` level (0 full, 1 medium, 2 low) based on average frame time.
  - **Config tables**: `DIFFICULTIES`, `THEMES` (sky palettes per stage), `ENEMY_TYPES` (grunt, interceptor, tank), `BOSSES` (six bosses), `KILLS_PER_STAGE`.
  - **Audio**: a Web Audio synth (`ensureAudio`, `beep`, `noiseBurst`, the `sfx` object). Sound is created in code, with no audio files.
  - **Input**: keyboard, mouse, and touch joystick handlers.
  - **Game state**: `resetGame`, `update(dt)`, `draw()`, and the `loop(now)` driver run on `requestAnimationFrame`.
  - **Weapons and power-ups**: `fireWeapon` (vulcan and laser, each with levels 1 to 3), `applyPowerUp` (`life`, `bomb`, `weapon`), `useBomb`.
  - **Bosses**: `BOSS_PATTERNS` (an array of attack-pattern sequences), `updateBoss`, `killBoss`.
  - **Persistence**: `localStorage` keys prefixed `skyfighter.` (`bestScore`, `muted`, `difficulty`, `reducedMotion`).

## Conventions

- Keep everything in `index.html` unless the user asks for a split. Keep the no-dependency design.
- Match the existing style: `const`/`let`, 2-space indentation, single quotes, arrow functions for callbacks, and short section banner comments.
- Game tuning values (speeds, spawn rates, HP, scores) are numeric literals in the config tables near the top of the script. Change them there rather than scattering new constants through the logic.
- Wrap `localStorage` access in a way that tolerates it being unavailable, as the existing code does for the best score. Do not add new storage keys without a `skyfighter.` prefix.
- Preserve accessibility hooks when changing UI: the `srLive` live region (`announce()`), the reduced-motion toggle, and the keyboard controls (arrows/WASD, Space to fire, X or Shift for a bomb, P or Escape to pause).
- Preserve responsive scaling: the canvas is a fixed 480x720 logical size (`BASE_W`, `BASE_H`) scaled via `resize()`.
- Keep the frame-rate-independent update style: `update(dt)` takes seconds, and `dt` is clamped to 0.05.

## Verification

Since there is no automated test suite, verify changes by opening `index.html` in a browser and playing through the affected path: start, fire, bombs, power-ups, stage transitions, bosses, pause, game over, best-score persistence, and both touch and keyboard input where relevant. Watch the browser console for errors. For visual or layout changes, check narrow (phone-width) and desktop viewports.

## Git

- Develop on the branch the task specifies. Do not push to other branches without permission.
- Write clear, descriptive commit messages that explain why the change was made.
