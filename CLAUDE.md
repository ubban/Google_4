# CLAUDE.md

Guidance for Claude Code when working in this repository.

## What this project is

Google_4 is a single-file browser game, **Sky Fighter**: a Raiden-style vertical shoot-'em-up with weapons, smart bombs, power-ups, and stage bosses. The entire game (HTML, CSS, and JavaScript) lives in `index.html`. `README.md` is a one-line stub.

## Repository layout

- `index.html` (~1,670 lines): the whole game.
  - `<style>` block: layout, HUD, overlays, and touch controls. Dark color scheme (`color-scheme: dark`).
  - Markup: canvas, HUD, start/pause overlays, touch joystick and FIRE/BOMB buttons.
  - One IIFE `<script>` containing all game logic.
- `README.md`: project title only.

There is no package manager, build step, bundler, test suite, linter, or CI configuration. Do not add one unless asked.

## Running and testing

- Open `index.html` directly in a browser, or serve the directory with any static server (for example `python3 -m http.server`).
- There are no automated tests. Verify changes by playing the game in a browser. Check the start, pause, and game-over flows, and the affected mechanic. Check both keyboard and touch controls when changing input handling.

## Code map inside the script

Key sections of the `<script>` IIFE, in roughly file order:

- **Persistence keys**: `STORAGE_BEST`, `STORAGE_MUTE`, `STORAGE_DIFF`, `STORAGE_REDUCED` (all prefixed `skyfighter.`). Read through `localStorage`.
- **Tuning tables**:
  - `DIFFICULTIES` (`easy` / `normal` / `hard`): spawn rate, enemy speed, enemy fire chance, lives, bombs, boss HP multiplier.
  - `KILLS_PER_STAGE`, `THEMES`, `ENEMY_TYPES`, `BOSSES`.
  - `BOSS_PATTERNS`: boss bullet patterns (radial, spread, spiral, cross, homing, double ring).
- **Audio**: `ensureAudio`, `beep`, `noiseBurst`, and the `sfx` helpers. Audio is created lazily on the first user gesture.
- **Input**: keyboard (`keys` map), mouse on the canvas, and touch joystick and buttons. Controls are documented in the `#controlsHint` element.
- **Game state**: `resetGame()` initializes state. `update(dt)` advances simulation. `draw()` renders. `loop(now)` drives both through `requestAnimationFrame`.
- **Gameplay systems**: `spawnEnemy`, `spawnBoss`, `updateBoss`, `killBoss`, `applyPowerUp`, `fireWeapon`, `useBomb`, `loseLife`, `endGame`, and combo/score helpers (`addScore`, `breakChain`).
- **Adaptive performance**: `trackPerf(dt)` runs every frame.

## Conventions

- Keep the game in one file unless the user asks to split it. Keep new code in the existing IIFE and follow its style: `const` by default, two-space indentation, and arrow functions for callbacks.
- Put tuning values in the existing config tables (`DIFFICULTIES`, `BOSSES`, `ENEMY_TYPES`) rather than hard-coding them in logic.
- Use the existing `STORAGE_*` keys with the `skyfighter.` prefix for any new persisted setting. Wrap storage access so the game still works if `localStorage` is unavailable.
- Preserve mobile support: keep `touch-action`, safe-area insets, and the responsive `resize()` behavior intact. The game renders at a base size of 480×720 and scales to fit.
- Respect the reduced-motion setting (`reducedMotion`) for new screen effects such as shake or flashes.
- Keep the game playable with the mouse, keyboard, and touch.

## Git

- Develop on the branch assigned for the session. Do not push to other branches without permission.
- Commit only when asked. Keep commits focused on one change.
- Do not create pull requests unless the user explicitly asks for one.
