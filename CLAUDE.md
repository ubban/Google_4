# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo is **Sky Fighter**, a single-file, browser-based vertical shoot-'em-up (Raiden-style) written in plain HTML, CSS and JavaScript. There is no build step, no package manager, no bundler, and no test suite.

Files:

- `index.html` — the entire game: markup (`<body>` HUD, overlays, touch controls), inline `<style>`, and one inline `<script>` IIFE containing all game logic.
- `README.md` — minimal project readme.

## Running the game

No install or build is needed. Open `index.html` directly in a browser, or serve the directory with any static server (e.g. `python3 -m http.server`) and visit it.

## Code structure (inside `index.html`)

The script is one `(function () { 'use strict'; ... })();` closure. Major sections, in file order:

- **DOM references** — `getElementById` lookups at the top of the IIFE.
- **Storage keys** — `skyfighter.*` keys for best score, mute, difficulty, and reduced motion (`localStorage`).
- **Adaptive performance** — `trackPerf` samples frame times and steps a `quality` level (0–2) up or down.
- **Config tables** — `DIFFICULTIES` (spawn rate, enemy speed/fire chance, lives, bombs, boss HP multiplier), `THEMES` (per-stage sky palettes), `ENEMY_TYPES` (grunt, interceptor, tank), `BOSSES` (six boss definitions), `KILLS_PER_STAGE`.
- **Audio** — `ensureAudio`, `beep`, `noiseBurst`, and the `sfx` helpers. Audio is created lazily on first user gesture.
- **HUD and effects** — `renderHearts`, `renderBombs`, `flashHit`, `showComboPop`, `showStageBanner`, `screenShake`, `announce` (screen-reader live region).
- **Input** — keyboard listeners, touch joystick, fire and bomb buttons, and mouse steering on the canvas.
- **Game state** — `resetGame`, `update(dt)`, `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`, `applyPowerUp`, `fireWeapon`, `useBomb`, `loseLife`, `endGame`, `togglePause`.
- **Boss patterns** — `BOSS_PATTERNS` and the `fire*` helpers (radial, spread, columns, spiral, cross, homing, double ring) used by `updateBoss`.
- **Rendering** — `draw()` and the per-entity draw helpers (`drawPlane`, `drawBossShip`, `drawClouds`).
- **Main loop** — `loop(now)` clamps `dt` to 0.05 s, calls `update` then `draw` while running and unpaused, and reschedules with `requestAnimationFrame`.

Game-state variables are plain `let` bindings in the closure; there are no modules, classes, or frameworks.

## Conventions

- Keep everything in `index.html` unless the user asks for a split. Do not introduce a build tool, framework, or dependency without asking.
- Match the existing style: 2-space indentation, single quotes in JS, `const` for constants and `let` for mutable state, arrow functions for event handlers, and `UPPER_SNAKE_CASE` for config tables.
- Group new code under the existing `// ---------- Section ----------` comment headers, or add a new header when a new area is introduced.
- Game tuning values live in the config tables near the top of the script. Prefer changing those over hard-coding numbers in logic.
- Use the `skyfighter.*` prefix for any new `localStorage` key, and wrap storage access so the game still works when storage is unavailable.
- Preserve accessibility hooks: keep `aria-live` announcements via `announce()` for significant events, and honor the reduced-motion setting when adding screen shake, flashes, or pulses.
- Keep the layout working on both desktop and touch devices (the `isTouchDevice` path enables on-screen controls).

## Controls (for reference when testing)

- Move: arrow keys or WASD; mouse also steers when hovering the canvas.
- Fire: Space (hold), or hold the mouse button on the canvas, or the FIRE button on touch.
- Bomb: X or Shift, or the BOMB button.
- Pause: P or Escape.
- Mute, reduced-motion, and pause are also available as top-bar buttons.

## Verification

There are no automated tests or linters configured. When changing game logic:

1. Load `index.html` in a browser and check the browser console for errors.
2. Play through at least one stage on each difficulty if the change touches spawning, bosses, or scoring.
3. Check the touch layout at a phone-sized viewport if the change touches the HUD or controls.

If you add tests or tooling, update this file with the commands to run them.

## Git

- Default development branch for this work is `claude/gifted-tesla-roqhc0`. Do not push to `main` without explicit permission.
- Write clear, descriptive commit messages that explain the gameplay or code change.
