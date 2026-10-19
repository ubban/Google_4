# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo is **Sky Fighter**, a Raiden-style vertical shoot-em-up built as a single-file HTML5 canvas game.

- `index.html` contains all of the markup, CSS (`<style>`), and game logic (one `<script>` IIFE). Everything lives here.
- `README.md` is a placeholder with no setup or run instructions.

There is **no build step, package manifest, test suite, or linter**. Do not invent `npm` scripts or test commands. To run the game, open `index.html` in a browser (or serve the directory with any static server).

## Layout of `index.html`

Top to bottom:

1. `<head>`: inline SVG favicon, then the `<style>` block (HUD pills, glassmorphic overlay card, touch controls, animations, `reduced-motion` styles).
2. `<body>`: HUD elements, overlay (title, difficulty buttons, stats row, start button), touch controls, and `<canvas id="gameCanvas">`.
3. `<script>`: a single `(function () { 'use strict'; ... })()` that contains the whole game. Its sections, in order:
   - DOM lookups and storage keys (`STORAGE_*`)
   - Adaptive performance (`quality` 0–2, based on rolling frame time)
   - Tuning tables: `DIFFICULTIES`, `THEMES`, `ENEMY_TYPES`, `BOSSES`, `KILLS_PER_STAGE`
   - Audio (`ensureAudio`, `beep`, `noiseBurst`, and the `sfx` object, all Web Audio)
   - HUD and feedback helpers (hearts, bombs, combo pop, stage banner, screen shake, flash)
   - Input (keyboard, touch joystick and buttons, mouse aim)
   - Game state (`resetGame`), rendering (`drawPlane`, `drawBossShip`, `draw`), and spawning
   - Simulation: `update(dt)`, power-ups, weapons (`fireWeapon`, `fireRadial`, and so on), boss patterns (`BOSS_PATTERNS`, `updateBoss`), `loseLife`, `endGame`, `togglePause`
   - `loop(now)`, which calls `update` then `draw`, with `dt` clamped to 0.05 s, and the start button wiring

## Conventions

- **Match the existing style.** Use `const`/`let`, arrow functions for callbacks, two-space indentation, and single quotes in JS. Section headers use `// ---------- Name ----------`.
- **Keep everything in one file** unless the user asks for a split. Do not add a bundler, framework, or dependencies without asking.
- **Tuning values live in the constant tables** (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `THEMES`, `BOSS_PATTERNS`). Prefer editing those over hard-coding numbers in logic.
- **Frame-rate independence:** movement and timers should use `dt` (seconds), not per-frame counts, unless the surrounding code already uses frame counts (`frame++` is used for pattern timing).
- **Persistence** uses `localStorage` with keys prefixed `skyfighter.` (`bestScore`, `muted`, `difficulty`, `reducedMotion`). Wrap new storage access in the same simple get/set pattern already used; do not add new keys without a reason.
- **Accessibility:** announce important state changes through `announce()` (the `aria-live` region). Respect `reducedMotion` for screen shake and flashes.
- **Coordinate system:** the logical playfield is `BASE_W` × `BASE_H` (480 × 720). `resize()` scales the canvas to fit the window, so gameplay code should use logical coordinates.

## Controls (for manual testing)

- Move: arrow keys or WASD. Mouse: move over the canvas to steer. Touch: left-side joystick.
- Fire: hold Space, or hold the mouse button on the canvas, or hold the fire button (all set `firing`/`keys.Space`).
- Bomb: `X` or Shift, or the bomb button. Bombs clear the screen and damage bosses.
- Pause: `P` or `Escape`, or the pause button.
- Mute and reduced-motion toggles are in the HUD. Difficulty can be changed on the title screen or while paused, but not during an active game (`update` is guarded by `running && !paused`).

## Verifying changes

Since there is no automated test suite:

1. Open `index.html` in a browser, or serve the folder statically, and check the browser console for errors.
2. Exercise the affected path: start a game, play through a stage transition (`KILLS_PER_STAGE`), trigger a boss fight, use a bomb, pause and resume, lose all lives, and confirm the best score persists after reload.
3. Check the touch layout at phone width if the change touches HUD or controls.

For logic-only changes, a quick syntax check of the extracted script is worthwhile, for example `node --check` on the script contents.

## Git

- Develop on the branch assigned for the task (currently `claude/gifted-tesla-51i8f4`). Do not push to other branches without permission.
- Write clear commit messages describing the gameplay or UI change.
- Do not open a pull request unless explicitly asked.
