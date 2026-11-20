# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo contains **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs in the browser. The entire game is a single self-contained file, `index.html`: markup, CSS (in a `<style>` block) and JavaScript (in one `<script>` block wrapped in an IIFE). There is no build step, no package manager, no framework and no test suite.

Files:

- `index.html`: the whole game.
- `README.md`: a one-line placeholder.
- `CLAUDE.md`: this file.

## Running the game

Open `index.html` directly in a browser, or serve the directory with any static server (for example `python3 -m http.server`) and visit it. No install step is needed.

## Code structure (`index.html`)

- **Head / meta**: viewport is locked (`user-scalable=no`, `viewport-fit=cover`) for mobile play. The favicon is an inline SVG data URI.
- **CSS**: dark color scheme (`color-scheme: dark`). HUD elements (`#hud`, `.stat`, `#hearts`, `#bombs`), overlays, vignette/flash effects and touch controls are styled here.
- **Markup**: a single `<canvas id="gameCanvas">` plus HUD, overlay (start/pause/game-over screens), and touch controls (`#touchControls`, `#moveZone` joystick, `#fireBtn`, `#bombBtn`).
- **Script** (one IIFE, `'use strict'`):
  - DOM references are grabbed at the top (`canvas`, `ctx`, `overlay`, HUD elements, buttons).
  - Persistent settings use `localStorage` with keys prefixed `skyfighter.` (`bestScore`, `muted`, `difficulty`, `reducedMotion`). Always guard assumptions about these values: they may be missing or malformed.
  - `DIFFICULTIES` holds per-difficulty tuning (lives, spawn timing, etc.). Read values from it rather than hard-coding them.
  - Audio is synthesized with the Web Audio API (`sfx`, `beep`, `noiseBurst`, `ensureAudio`). Audio must be started from a user gesture.
  - `resize()` scales the canvas, using a base resolution of `BASE_W = 480` by `BASE_H = 720`.
  - Game state (`score`, `lives`, `running`, `paused`, `gameOver`, etc.) is module-level and reset by `resetGame()`.
  - Entities are spawned by `spawnEnemy()`, `spawnBoss()` and `maybeSpawnPowerUp()`. Boss attack patterns are small functions (`fireRadial`, `fireAimedSpread`, `fireSpiralBurst`, `fireCross`, `fireHomingSeeded`, `fireDoubleRing`, etc.) driven by `updateBoss()`.
  - `update(dt)` advances the simulation and `draw()` renders it. `loop(now)` is the `requestAnimationFrame` driver.
  - Input: keyboard (`keys` map keyed by `e.code`), touch joystick and buttons, and mouse on the canvas.

## Controls (for reference when changing input code)

- Move: arrow keys or WASD; touch joystick on mobile.
- Fire: Space (held), the fire button, or mouse button held down on the canvas (the mouse also steers the ship toward the cursor).
- Bomb: X or Shift, or the bomb button.
- Pause: P or Escape.

## Conventions

- Keep the game dependency-free: plain HTML, CSS and vanilla JavaScript. Do not add a bundler, framework or npm dependency unless the user asks for it.
- Keep changes inside `index.html` unless there is a clear reason to split files. If you do split files, update this section.
- Preserve the IIFE and `'use strict'` wrapper, and keep new top-level helpers inside it.
- Match the existing style: 2-space indentation, single quotes in JS, `const` by default, and descriptive function names.
- Prefer `textContent` for HUD text. `innerHTML` is used only for the hearts and bombs icon rows (`renderHearts`, `renderBombs`); never pass player-controlled or other dynamic strings into it.
- Keep the game playable on touch devices and at phone widths. Test any HUD or layout change at a narrow viewport as well as desktop.
- Respect the `reducedMotion` setting when adding screen shake, flashes or other motion effects.
- Keep the `skyfighter.` localStorage key prefix for any new persisted setting.

## Testing and verification

There is no automated test suite. To verify a change:

1. Open `index.html` in a browser (or serve it locally).
2. Start a game, then check movement, firing, bombs, pause/resume, lives and game over.
3. Check the best-score and difficulty persistence across a page reload.
4. For layout or input changes, check both a desktop viewport and a narrow mobile viewport.
5. Check the browser console for errors.

If you add tests, document how to run them here.

## Git workflow

- Develop on the branch named by the environment (currently `claude/gifted-tesla-rjqk7n`). Do not push to other branches without permission.
- Commit only when asked. Write clear commit messages describing the gameplay or code change.
- Do not create pull requests unless the user explicitly asks for one.
