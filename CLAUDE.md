# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo contains **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs as a single-page HTML5 canvas app.

- `index.html` is the entire game: markup, CSS (`<style>` block, ~lines 10–362), and JavaScript (`<script>` block, ~lines 422–1669). There is no build step, bundler, package manager, or test suite.
- `README.md` is a placeholder and does not describe the game.
- Git history: `4efdad1` (initial commit), `c4dc8f6` (adds Sky Fighter).

## Running and verifying

- Open `index.html` directly in a browser, or serve the directory with any static server (e.g. `python3 -m http.server`).
- There are no automated tests or linters configured. Verify changes by playing the game in a browser, checking the browser console for errors, and exercising the feature you touched on both desktop (keyboard) and touch layouts.

## Code structure (inside the main IIFE)

The script is wrapped in a single `(function () { 'use strict'; ... })();` IIFE. Nothing is exported to `window`. Keep new code inside it.

Key areas, roughly in file order:

- **DOM refs**: `const` lookups by element id at the top of the IIFE. Add new HUD elements here.
- **Tuning tables**: `DIFFICULTIES` (spawn rates, enemy speed, shoot chance, lives, bombs, boss HP multiplier), `KILLS_PER_STAGE`, `THEMES`, `ENEMY_TYPES`, `BOSSES`. Balance changes should usually go here rather than in logic.
- **Persistence**: `localStorage` keys prefixed `skyfighter.` (`bestScore`, `muted`, `difficulty`, `reducedMotion`). Reads and writes are not wrapped in try/catch; if you add storage use, keep the same key prefix and handle missing values with a default.
- **Audio**: Web Audio helpers (`ensureAudio`, `beep`, `noiseBurst`). Audio is created lazily on the first user gesture (`ensureAudio()` is called from the start button); don't create an `AudioContext` on load.
- **Input**: keyboard via `keys[e.code]` on window `keydown`/`keyup`; touch via a virtual joystick (`joystickStart/Move/End`) and fire/bomb buttons; mouse/pointer mapped through `canvasPointFromEvent`.
- **Layout**: `BASE_W = 480`, `BASE_H = 720` logical resolution; `resize()` scales the canvas to fit the viewport.
- **Game state and update**: `resetGame()` initialises state; `update(dt)` advances entities, collisions, spawns, and stage progression; `updateBoss(dt)` drives boss patterns (`BOSS_PATTERNS`, `fire*` helpers); `applyPowerUp`, `useBomb`, `fireWeapon`, `loseLife`, `killBoss`, `endGame`, `togglePause`.
- **Rendering**: `draw()` renders everything each frame (clouds, planes, bosses, bullets, power-ups, particles, HUD effects). `drawPlane` and `drawBossShip` handle sprites.
- **Main loop**: `loop(now)` clamps `dt` to 0.05 s, calls `trackPerf`, then `update` and `draw` when running and not paused. It is started by a single `requestAnimationFrame(loop)` at the end.

## Conventions

- **Single file**: keep everything in `index.html` unless the user asks for a split. Don't introduce a build system or dependencies without asking.
- **Plain JS, ES2015+**: match the existing style — `const`/`let`, arrow functions for callbacks, two-space indentation, single quotes.
- **Units**: positions and sizes are in logical canvas pixels (480×720); `dt` is in seconds.
- **Accessibility**: the code uses an `srLive` live region via `announce(text)` for screen-reader updates, and a reduced-motion setting that gates screen shake and similar effects. New announcements and motion effects should respect both.
- **Mobile**: touch controls must keep working; check the layout at phone widths when changing HUD or overlay markup.
- **Pause/restart**: state changes that affect gameplay should respect `running` and `paused` flags so pausing and the overlay keep working.

## Working in this repo

- Develop on the branch assigned for the session (currently `claude/gifted-tesla-3z1cfa`). Don't push to other branches without explicit permission.
- Don't open a pull request unless the user asks for one.
- Keep commits focused. Commit messages should describe the gameplay or code change, not the tooling used.
