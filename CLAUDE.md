# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo contains **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs in the browser on a 2D `<canvas>`. The entire game is one self-contained file:

- `index.html` (~1670 lines): inline `<style>`, markup, and one inline `<script>` IIFE.
- `README.md`: placeholder only. It has no setup or usage instructions.

There is no build step, no package manager, no bundler, no test suite, and no linter configured. Do not add any of these unless the user asks.

## Running the game

Open `index.html` directly in a browser, or serve the directory with any static server, for example `python3 -m http.server`. No install is needed.

## Code layout inside `index.html`

The script is wrapped in a single `(function () { 'use strict'; ... })();` block, so nothing leaks to `window`. Keep new code inside that wrapper.

Roughly in order, the script contains:

1. **DOM references**: `const xxxEl = document.getElementById(...)`. Every HUD, overlay, and touch control is looked up once here.
2. **Storage keys**: `STORAGE_BEST`, `STORAGE_MUTE`, `STORAGE_DIFF`, `STORAGE_REDUCED`. All use the `skyfighter.*` prefix.
3. **Config tables**:
   - `DIFFICULTIES` (`easy` / `normal` / `hard`): spawn rate, enemy speed, shoot chance, lives, bombs, boss HP multiplier.
   - `KILLS_PER_STAGE`: kills needed to advance a stage.
   - `THEMES`: sky gradient, cloud and sun colors, and a name for each stage background. Cycled per stage.
   - `ENEMY_TYPES`: `grunt`, `interceptor` (weaves), `tank` (4 HP, unlocks at stage 2).
   - `BOSSES`: six bosses, each with colors and an `hpMul`.
   - `BOSS_PATTERNS`: bullet-pattern functions (radial, spread, columns, spiral, cross, homing, double ring).
4. **Audio**: `ensureAudio()` lazily creates the `AudioContext` on first user gesture. `beep()`, `noiseBurst()`, and the `sfx` object synthesize every sound. There are no audio files.
5. **Input**: a keyboard map on `keys[e.code]`, and touch joystick plus fire and bomb buttons. Touch controls are shown only on touch devices.
6. **Game state and logic**: `resetGame()`, `update(dt)`, `spawnEnemy()`, `spawnBoss()`, `updateBoss(dt)`, `applyPowerUp()`, `useBomb()`, `loseLife()`, `endGame()`, `togglePause()`.
7. **Rendering**: `draw()`, `drawPlane()`, `drawBossShip()`, `drawClouds()`. All drawing is immediate-mode canvas calls.
8. **Main loop**: `loop(now)` runs on `requestAnimationFrame`. It clamps `dt` to 50 ms and calls `trackPerf`, `update`, and `draw`.

### Key conventions

- **Time is in seconds.** `dt` is seconds per frame. Speeds are per-second values multiplied by `dt`.
- **Logical resolution is 480×720.** `BASE_W` and `BASE_H` define it, and `resize()` scales the canvas to fit the window. Do not hardcode screen pixels.
- **Adaptive quality.** `trackPerf` watches the rolling average frame time and steps `quality` between 0 (full), 1 (medium), and 2 (low). Respect this when adding expensive effects.
- **Reduced motion.** Honor `reducedMotion` (from the OS preference or the in-game toggle). Gate screen shake and heavy particle effects behind it.
- **Accessibility.** Announce important game events through `announce()`, which writes to the `srLive` live region. Keep HUD text in real DOM elements, not only in the canvas.
- **Persistence.** Only `localStorage` is used, and only for best score, mute, difficulty, and reduced motion. Do not add other storage without asking.

## Controls (for reference when testing)

| Action | Keyboard | Touch |
|---|---|---|
| Move | Arrow keys or WASD | Left-side joystick |
| Fire | Space (hold) | Fire button (hold) |
| Bomb | X or Shift | Bomb button |
| Pause | P or Esc | Pause button |

Mouse: moving over the canvas steers the ship, and mouse-down fires.

## Making changes

- **Keep it a single file** unless the user explicitly asks for a split. Preserve the IIFE wrapper and `'use strict'`.
- **Match the surrounding style.** The code uses 2-space indentation, single quotes, `const` by default, arrow functions for callbacks, and short `// ----------` section headers. Follow the same pattern for new sections.
- **Add new content through the existing tables.** New enemies, bosses, themes, or difficulty settings should be new entries in `ENEMY_TYPES`, `BOSSES`, `THEMES`, or `DIFFICULTIES`, not special cases in the loop.
- **Balance numbers live in the config tables.** Prefer tuning those over changing logic.
- **Sound is synthesized.** If you add a sound, add it as an `sfx` entry built from `beep` or `noiseBurst`. Do not introduce audio asset files.
- **Verify in a browser.** There are no automated tests, so load the page and play it after a change. Check desktop keyboard controls, and check touch layout at phone width if the change touches the HUD or controls.

## Git

- Develop on the branch the session specifies (currently `claude/gifted-tesla-jcrvy9`). Do not push to other branches without permission.
- Write clear, descriptive commit messages.
- Do not open a pull request unless the user asks for one.
