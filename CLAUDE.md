# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo contains **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs in the browser. It is a single self-contained file:

- `index.html` (~1,670 lines): inline CSS in `<style>`, game logic and rendering in one `<script>` IIFE, drawn on a `<canvas id="gameCanvas">`. No build step, no dependencies, no bundler.
- `README.md`: minimal placeholder.

There is no package.json, test suite, linter, or CI configuration.

## Running the game

Open `index.html` directly in a browser, or serve the directory with any static server (e.g. `python3 -m http.server`) and visit it. No install step is needed.

## Code layout inside `index.html`

- **Markup**: HUD elements (score, best, multiplier, weapon, hearts, bombs), overlay/start screen, pause and mute/motion buttons, touch controls (joystick, fire, bomb).
- **Configuration constants** (in the script, near the top):
  - `STORAGE_*`: `localStorage` keys (`skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion`).
  - `DIFFICULTIES`: `easy` / `normal` / `hard` tuning (spawn rate, enemy speed, shoot chance, lives, bombs, boss HP multiplier).
  - `KILLS_PER_STAGE`, `THEMES` (per-stage sky palettes and names).
- **Audio**: lazily created `AudioContext` (`ensureAudio`), synthesized sounds via `beep` and `noiseBurst`. Respects the mute toggle.
- **Input**: keyboard handlers on `window`, touch joystick and buttons, and mouse on the canvas. Reduced-motion and device-motion settings are handled in the UI-control section.
- **Game state and loop**: `resetGame`, `update(dt)` (movement, firing, collisions, spawning, power-ups, boss logic), `draw()`, and `loop(now)` driven by `requestAnimationFrame`.
- **Enemies and bosses**: `spawnEnemy`, `spawnBoss`, `updateBoss`, `killBoss`. Boss attack patterns are the `fire*` helpers (radial, aimed spread, columns, spiral, cross, homing, double ring).
- **Flow**: `loseLife`, `endGame`, `togglePause`, `resize`.

## Controls (for reference when changing input code)

- Move: arrow keys or A/D (also mouse on canvas, touch joystick).
- Fire: Space (hold), or hold the on-screen fire button.
- Bomb: X or Shift, or the on-screen bomb button.
- Pause: P or Escape.

## Conventions

- Keep the game as a single HTML file unless the user asks for a restructure. Do not add a build system or external dependencies without a clear reason.
- Stick to the existing style: `const`/`let`, 2-space indentation, functions declared inside the IIFE, and the `DIFFICULTIES`/`THEMES` tables for tunable values rather than hard-coded magic numbers.
- Persistent player data goes through the existing `STORAGE_*` keys. Wrap new `localStorage` access in try/catch, since storage may be unavailable (private windows, blocked site data).
- Preserve mobile behavior: the layout uses `viewport-fit=cover`, safe-area insets, and `touch-action: none`. Check phone-width layouts when changing HUD or touch controls.

## Verification

There are no automated tests. After changing game logic or rendering, load `index.html` in a browser and play through at least:

1. Starting a run, moving, firing, and using a bomb.
2. Losing a life and reaching game over; confirm the best score persists after reload.
3. Pausing and resuming, and switching difficulty.
4. Touch controls, if the change touches input or layout.

## Git

- Development branch for this work: `claude/gifted-tesla-b5wrep`. Commit and push there; do not push to other branches without permission.
- Do not open a pull request unless the user explicitly asks for one.
