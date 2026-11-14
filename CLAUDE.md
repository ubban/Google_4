# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a single-page browser game, **Sky Fighter**: a Raiden-style vertical shoot-'em-up rendered on a `<canvas>`. The whole game lives in one file.

- `index.html` (~1,670 lines): HTML, inline CSS (`<style>`, ~lines 10–360), markup (`<body>`, ~line 361), and one inline `<script>` IIFE (~lines 422–1669) containing all game logic.
- `README.md`: a placeholder title. It has no setup or usage instructions.

There is no build step, no package manager, no test suite, and no linter. Nothing needs to be installed.

## Running and verifying

- Open `index.html` directly in a browser, or serve the directory with any static server (for example `python3 -m http.server`).
- There are no automated tests. Verify changes by playing the game in a browser. For changes to the game loop, check at least: start, movement, firing, a bomb, pause/resume, a stage transition, a boss fight, and game over.
- Keep the game working without a build. Do not add a bundler, framework, or dependencies unless the user asks for one.

## Architecture (inside `index.html`)

The script is one `'use strict'` IIFE. State and functions are closure-scoped, and nothing is exported to `window`.

Main pieces, in file order:

- **DOM references**: looked up once at the top of the IIFE by element ID (`gameCanvas`, `overlay`, `startBtn`, HUD elements, touch controls, boss bar). Add a new element ID here and in the markup together.
- **Persistence**: `localStorage` keys prefixed `skyfighter.` (`bestScore`, `muted`, `difficulty`, `reducedMotion`). Read these defensively; they can be missing or hold unexpected values.
- **Tuning tables**:
  - `DIFFICULTIES` (`easy` / `normal` / `hard`): spawn timing, enemy speed, shoot chance, lives, bombs, boss HP multiplier.
  - `KILLS_PER_STAGE` (18): kills needed to advance a stage.
  - `THEMES`: six sky palettes, cycled per stage.
  - `ENEMY_TYPES`: `grunt`, `interceptor` (weaving), `tank` (4 HP, appears from stage 2).
  - `BOSSES`: six bosses with colors and HP multipliers.
  - `BOSS_PATTERNS` (~line 1372): bullet-pattern functions (radial, aimed spread, columns, spiral, cross, homing, double ring).
- **Audio**: `ensureAudio()` creates the `AudioContext` lazily, after a user gesture. `beep()` and `noiseBurst()` synthesize all sound effects. There are no audio files.
- **Input**: keyboard (`keys[e.code]`), touch joystick and buttons, and mouse. Keyboard bindings are at ~line 1153:
  - Move: arrows or WASD
  - Fire: Space (or hold the fire button)
  - Bomb: X or Shift
  - Pause: P or Escape
- **Layout**: `resize()` scales the fixed `BASE_W × BASE_H` (480 × 720) logical canvas to the viewport.
- **Game state**: module-level `let` variables (`score`, `lives`, `player`, arrays of enemies, bullets, power-ups, and so on). `resetGame()` reinitializes them.
- **Update / draw**: `update(dt)` advances the simulation. `draw()` renders it. `loop(now)` is the `requestAnimationFrame` driver. It clamps `dt` to 0.05 s and skips update and draw when paused or not running.
- **Adaptive quality**: `trackPerf()` watches frame times and steps a `quality` level (0–2) up or down. Drawing code reads this level to reduce effects.
- **Reduced motion**: honors `prefers-reduced-motion` and a user toggle. It disables screen shake and flashes.
- **Accessibility**: `announce()` writes to a live region (`srLive`) for screen readers.

## Conventions

- Keep everything in `index.html` unless there is a clear reason to split it. If you split it, update this file.
- Match the existing style: 2-space indent, single quotes in JS, `const` by default, `let` for mutable game state, and small named functions.
- Put tuning numbers in the existing constant tables rather than scattering magic numbers through `update()` and `draw()`.
- Respect the reduced-motion and quality settings when adding visual effects.
- Keep the touch controls working when you change input handling. The game is played on phones too.
- Do not add external network requests, CDN scripts, or tracking.

## Git

- Development branch for this work: `claude/gifted-tesla-rcqs3t`. Commit and push there.
- Do not open pull requests unless the user asks.
- Commit history is short (`Initial commit`, then the Sky Fighter addition). Write commit messages in the same plain, descriptive style.
