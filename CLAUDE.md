# CLAUDE.md

Guidance for working in this repository.

## Project overview

Sky Fighter is a Raiden-style vertical shoot-'em-up that runs in the browser on an HTML5 canvas. The whole game is one self-contained file. There is no build step, no package manager, no framework, and no test suite.

## Repository layout

- `index.html` — the entire game:
  - Inline `<style>` in the head: the HUD, glassmorphic overlay card, touch controls, and animations.
  - Markup: the canvas (`#gameCanvas`), HUD elements, overlay, and touch controls.
  - One inline `<script>`, wrapped in an IIFE with `'use strict'`, containing all game logic and rendering.
- `README.md` — title only.

## Running

Open `index.html` directly in a browser, or serve the directory with any static file server. Nothing needs to be installed.

## Structure of the game script

The script runs top to bottom inside one IIFE. The main parts, in order:

1. **DOM references and storage keys.** Elements are looked up once at the top. Persisted settings use `localStorage` keys prefixed `skyfighter.` (`bestScore`, `muted`, `difficulty`, `reducedMotion`).
2. **Adaptive performance.** `trackPerf` samples frame times and steps a `quality` level (0–2) up or down.
3. **Config tables.** `DIFFICULTIES`, `THEMES`, `ENEMY_TYPES`, `BOSSES`, and `KILLS_PER_STAGE` hold the tunable numbers. Boss attack patterns are in `BOSS_PATTERNS`.
4. **Audio.** Sound is synthesized with the Web Audio API (`ensureAudio`, `beep`, `noiseBurst`). No audio files are loaded.
5. **Input.** Keyboard state is kept in a `keys` map keyed by `e.code`. Mouse and touch input use a joystick and buttons for mobile.
6. **Game state and actions.** `resetGame`, `spawnEnemy`, `spawnBoss`, `fireWeapon`, `useBomb`, `applyPowerUp`, `loseLife`, `endGame`, `togglePause`.
7. **Simulation.** `update(dt)` advances the game by one step. `updateBoss(dt)` and the `fire*` pattern helpers drive bosses.
8. **Rendering.** `draw()` renders each frame. `drawPlane`, `drawBossShip`, and `drawClouds` are the sprite helpers.
9. **Loop.** `loop(now)` computes `dt` in seconds, clamps it to 0.05, and calls `update` and `draw` only while the game is running and not paused.

## Conventions

- Vanilla JavaScript using `const`/`let` and arrow functions. Do not add modules, a bundler, or a framework unless the user asks for one.
- Keep the game in `index.html` unless the user asks for it to be split. Keep the IIFE and `'use strict'`.
- Movement and timers are time-based: multiply by `dt` in seconds. Do not tie speeds to frame counts.
- Functions use camelCase. Tunable constants use SCREAMING_SNAKE_CASE. DOM references are plain `const` values at the top of the script.
- Comments are sparse. Use short `// ---------- Section ----------` headers for major sections, and comment only where the logic is not obvious.
- Any new persisted setting uses the `skyfighter.` key prefix and follows the existing read/write pattern.

## Gameplay tuning

- Difficulty: `DIFFICULTIES` (spawn rates, enemy speed and fire chance, starting lives and bombs, boss HP multiplier).
- Pacing: `KILLS_PER_STAGE` controls how many kills advance a stage.
- Enemies: `ENEMY_TYPES` (size, HP, speed multiplier, score, the minimum stage they appear in).
- Bosses: `BOSSES` (name, colors, HP multiplier) and `BOSS_PATTERNS` (attack patterns).
- Visuals: `THEMES` (sky, cloud, and sun colors for each stage).

## Controls (for reference when testing)

- Mouse: move to steer, click to fire.
- Keyboard: arrows or WASD to move, Space to fire, X or Shift for a bomb, P or Esc to pause.
- Touch: drag on the left to fly, tap FIRE, tap BOMB.

## Verification

There is no test suite or linter configured. Before finishing a change:

- Check the script parses. For example, extract the inline script to a scratch file and run `node --check` on it.
- Load `index.html` in a browser and play through the affected feature. Check the desktop and a narrow mobile viewport, and check the touch controls if input code changed.
- Confirm that persisted settings still load and save correctly.
