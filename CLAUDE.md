# CLAUDE.md

Guidance for Claude Code when working in this repository.

## What this repo is

`Google_4` holds **Sky Fighter**, a single-page HTML5 canvas vertical shoot-em-up (Raiden-style). The whole game is one file, `index.html`: markup, CSS, and JavaScript inline. `README.md` is a placeholder.

There is no build step, package manager, test suite, or linter. Open `index.html` in a browser to run it.

## Layout

- `index.html` (~1,670 lines)
  - `<style>` (around lines 10–362): HUD, overlay card, touch controls, animations, reduced-motion styles.
  - `<body>` markup (from line 363): `#gameCanvas`, HUD elements, overlay, touch controls.
  - One `<script>` IIFE (from line 422, `'use strict'`) containing all game logic. Nothing is exported; state lives in closures.
- `README.md`: placeholder.

## Structure of the game script

The script is organized by `// ----------` section comments. Keep new code in the matching section:

- **Config tables** near the top: `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES` (per-stage sky palettes), `ENEMY_TYPES`, `BOSSES`.
- **Persistence**: `STORAGE_*` keys under the `skyfighter.*` prefix in `localStorage` (best score, mute, difficulty, reduced motion). Every access is a direct `localStorage` call, so guard new writes the same way if they can run in private modes.
- **Audio**: Web Audio helpers (`ensureAudio`, `beep`, `noiseBurst`). Audio starts only after a user gesture via `ensureAudio()`.
- **Adaptive performance**: `trackPerf` moves a `quality` level (0–2) based on frame time.
- **Input**: `keys` map keyed by `e.code`, touch joystick, and fire/bomb buttons. Mouse and touch both feed `canvasPointFromEvent`.
- **Gameplay**: `resetGame`, `update(dt)`, `spawnEnemy`, `spawnBoss`, `fireWeapon`, `useBomb`, `applyPowerUp`, `updateBoss`, `loseLife`, `endGame`, `togglePause`.
- **Boss patterns**: `BOSS_PATTERNS` plus `fire*` helpers (radial, spread, columns, spiral, cross, homing, double ring).
- **Rendering**: `draw()` and helpers (`drawPlane`, `drawBossShip`, `drawClouds`). `draw` runs only while the game is running and unpaused.
- **Loop**: `loop(now)` clamps `dt` to 0.05 s and calls `update` then `draw`. It is started once with `requestAnimationFrame`.

Controls (also shown in `#controlsHint`): mouse to steer and click to fire; arrows or WASD to move; Space to fire; X or Shift for a bomb; P or Esc to pause. Touch uses a left-side drag joystick plus FIRE and BOMB buttons.

## Conventions

- Match the existing style: 2-space indent, single quotes, `const` by default, arrow callbacks, and short comments that mark sections.
- Keep it a single self-contained file unless the user asks to split it. Don't add a bundler, framework, or external CDN dependency for a small change.
- Put tunable numbers in the config tables (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `THEMES`) rather than hard-coding them in logic.
- Rendering uses only canvas 2D. Respect the `reducedMotion` flag for new screen-shake, flash, or motion effects.
- Gameplay state resets in `resetGame()`. Any new per-run state must be reset there.
- The canvas uses a fixed logical size (`BASE_W` 480 × `BASE_H` 720) scaled by `resize()`. Coordinates in game logic use that logical space.

## Working on it

- There is nothing to install and nothing to run in CI. Verify changes by opening `index.html` in a browser, or with a headless Chromium if one is available, and playing through the affected path: menu, a stage transition, a boss fight, pause and resume, game over, and mobile layout when touch code changes.
- Check the browser console for errors after any change to the script.
- Before committing, confirm `git diff` touches only the intended lines. `index.html` is large, so avoid whole-file rewrites.
- Use `git push -u origin <branch>` for pushes. Don't open a pull request unless asked.
