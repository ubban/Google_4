# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo contains **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs in the browser as a single self-contained HTML file. There is no build step, no package manager, no framework, and no test suite.

Files:

- `index.html` (~1,670 lines): the entire game. CSS in a `<style>` block (starts ~line 10), markup in `<body>` (~line 361), and one IIFE in a `<script>` block (~line 422) that holds all game logic.
- `README.md`: a placeholder. It has no project documentation.

## Running and verifying

- Open `index.html` directly in a browser, or serve the directory with any static server (for example `python3 -m http.server`).
- There are no automated tests, linters, or build commands. Verify changes by playing the game in a browser. For touch-layout changes, check a narrow viewport or device emulation as well.

## Code architecture (inside the script IIFE)

The script is wrapped in `(function () { 'use strict'; ... })()` and keeps all state in module-level `let`/`const` variables. Major sections, in source order:

- **DOM references**: `getElementById` lookups at the top of the IIFE.
- **Persistence**: `localStorage` keys prefixed `skyfighter.` (`bestScore`, `muted`, `difficulty`, `reducedMotion`). Every read and write is at top level, with no try/catch.
- **Adaptive performance**: `trackPerf(dt)` samples the last 45 frames and moves `quality` between 0 (full), 1 (medium), and 2 (low). Particle and cloud density respond to it.
- **Tuning tables**: `DIFFICULTIES` (spawn rate, enemy speed, shot chance, lives, bombs, boss HP multiplier), `THEMES` (sky palettes, one per stage cycle), `ENEMY_TYPES` (grunt, interceptor, tank), `BOSSES` (six bosses with colors and HP multipliers), and `KILLS_PER_STAGE` (18).
- **Audio**: a lazily created Web Audio context (`ensureAudio`). Sound effects go through `beep`, `noiseBurst`, and the `sfx` object. Audio is created on first user gesture.
- **Canvas sizing**: fixed logical resolution `BASE_W × BASE_H` = 480 × 720, scaled to fit the window (capped at 1.6×) in `resize()`.
- **Input**: keyboard (`keys` map keyed by `KeyboardEvent.code`), mouse steering on the canvas, and a touch joystick plus fire and bomb buttons. Touch controls are shown only on touch devices.
- **Game state and logic**: `resetGame`, `update(dt)`, `fireWeapon`, `spawnEnemy`, `spawnBoss`, `updateBoss`, `killBoss`, `loseLife`, `endGame`, `togglePause`. Weapons are `vulcan` and `laser`. Power-ups are applied in `applyPowerUp`.
- **Boss attack patterns**: the `BOSS_PATTERNS` array of functions (radial, aimed spread, columns, spiral, cross, homing, double ring).
- **Rendering**: `drawPlane`, `drawBossShip`, `drawClouds`, and `draw()`. `draw()` is called only while the game is running and not paused.
- **Main loop**: `loop(now)` clamps `dt` to 50 ms, calls `trackPerf`, then `update` and `draw`, and reschedules with `requestAnimationFrame`.

Game flow: the overlay start button calls `resetGame`. Kills accumulate toward `KILLS_PER_STAGE`, which triggers a boss fight. Defeating the boss increments `stage`, which advances the theme and difficulty.

## Conventions

- Keep everything in `index.html` unless there is a clear reason to split it. The single-file layout is intentional.
- Stay with vanilla JavaScript and Canvas 2D. Do not add dependencies, a bundler, or a framework without an explicit request.
- Follow the existing style: 2-space indentation, single quotes in JS, `const` by default, and section headers written as `// ---------- Name ----------`.
- Put new tuning values in the existing config tables (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `THEMES`) rather than hard-coding them in logic.
- Keep the `localStorage` key prefix `skyfighter.` for any new persisted setting.
- Preserve the accessibility hooks: the `srLive` live region (`announce()`), the `reducedMotion` class on `<body>`, and the `prefers-reduced-motion` default.
- Keep the layout usable on phones (`viewport-fit=cover`, touch controls) and keep the desktop control hints in `#controlsHint` accurate when controls change.

## Controls (for reference when editing)

- Move: mouse (on canvas), Arrow keys or WASD, touch joystick
- Fire: Space, hold left mouse, or the FIRE button
- Bomb: X or Shift, or the BOMB button (`maxBombs` is set by difficulty)
- Pause: P or Esc

## Git workflow

- Develop on the branch named in the session instructions (currently `claude/gifted-tesla-eydklr`). Do not push to other branches without permission.
- Commit with clear messages and push with `git push -u origin <branch>`.
- Do not open a pull request unless explicitly asked.
