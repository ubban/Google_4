# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Sky Fighter is a Raiden-style vertical shoot-'em-up that runs in the browser. The whole game is a single self-contained file, `index.html`: inline CSS in `<style>`, markup in `<body>`, and one inline `<script>` (an IIFE with `'use strict'`). There is no build step, no bundler, no package manager, and no test suite.

Repository layout:

- `index.html` - the entire game (about 1,670 lines).
- `README.md` - a placeholder; it has no setup instructions.

## Running the game

Open `index.html` directly in a browser, or serve the directory with any static server, for example `python3 -m http.server`. Nothing needs to be installed.

## Code structure (inside `index.html`)

The script is organized into commented sections, in this order:

- **DOM lookups and constants**: element references at the top, `STORAGE_*` localStorage keys (`skyfighter.*`).
- **Adaptive performance** (`trackPerf`): averages frame time over 45 frames and steps a `quality` level (0 full, 1 medium, 2 low).
- **Tuning tables**: `DIFFICULTIES` (spawn rates, enemy speed and fire chance, lives, bombs, boss HP multiplier), `KILLS_PER_STAGE`, `THEMES` (sky palettes cycled by stage), `ENEMY_TYPES` (grunt, interceptor, tank), and `BOSSES` (six bosses, in order).
- **Audio**: a Web Audio synth (`ensureAudio`, `beep`, `noiseBurst`). No audio files are loaded.
- **HUD and feedback**: hearts, bombs, combo pop, stage banner, screen shake, hit and bomb flashes, and `announce()` for the screen-reader live region.
- **Layout and touch**: `resize()` scales a fixed `BASE_W` x `BASE_H` (480 x 720) play field to the viewport; a virtual joystick and fire/bomb buttons handle touch input.
- **Game state**: `resetGame()`, `update(dt)` (the main simulation), `applyPowerUp`, weapon firing (`fireWeapon`), bombs (`useBomb`), and boss attack patterns (`BOSS_PATTERNS`, `updateBoss`, `killBoss`).
- **Life and flow**: `loseLife()`, `endGame()`, `togglePause()`.
- **Rendering**: `draw()` and the helpers `drawPlane`, `drawBossShip`, `drawClouds`.
- **Main loop**: `loop(now)` clamps `dt` to 0.05 s, then calls `update` and `draw` while the game is running and not paused.

Controls: arrow keys or WASD to move (read through `keys[e.code]`), Space to fire, `X` or Shift for a bomb, `P` or Escape to pause. Touch devices get on-screen controls automatically.

Persistent settings live in `localStorage`: best score, mute, difficulty, and reduced motion. Reads and writes are not wrapped in try/catch today, so keep that in mind if you touch them (for example, in private browsing where storage can throw).

## Conventions

- Keep the game in one file unless the user asks for a split. Add new CSS to the `<style>` block and new logic to the existing script IIFE.
- Follow the file's existing style: two-space indentation, single quotes in JavaScript, `const` by default, arrow functions for callbacks, and a short `// ---------- Section ----------` comment before each new section.
- Put gameplay numbers in the existing tuning tables (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `BOSS_PATTERNS`) instead of hard-coding them in the update or draw code.
- Respect `reducedMotion` and the `prefers-reduced-motion` media query when you add visual effects such as shake, flashes, or pulses.
- Announce important state changes through `announce()` so screen-reader users get them.
- Keep the play field at its 480 x 720 logical size; scaling is handled in `resize()`.

## Verification

There are no automated tests or linters configured. To check a change:

1. Open `index.html` in a browser (or serve it locally) and check the browser console for errors.
2. Play through at least one stage and one boss on each difficulty you touched.
3. Check pause and resume (`P`/Escape), a game over, and the best-score update.
4. For touch changes, use a device emulator and check the joystick and fire/bomb buttons.
5. For performance or rendering changes, check the frame rate on a low-end setting, since the adaptive quality logic depends on frame timing.

A quick syntax check of the inline script is also possible with Node, for example by extracting the `<script>` block to a temporary file and running `node --check` on it.

## Git

Development happens on feature branches. Commit with clear, descriptive messages and push only to the branch the task names. Do not open a pull request unless the user asks for one.
