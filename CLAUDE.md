# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Google_4 is a single-file browser game, **Sky Fighter**: a Raiden-style vertical shoot-'em-up. The player switches weapons, drops smart bombs, and fights stage bosses.

The whole game lives in `index.html` (about 1,800 lines): inline CSS in a `<style>` block, markup for the HUD and overlays, and one `<script>` IIFE with the game logic. There is no build step, no bundler, no package manager, and no test suite. `README.md` is a one-line placeholder.

## Repository layout

- `index.html` - the entire game (markup, styles, and logic).
- `README.md` - project title only.
- `CLAUDE.md` - this file.

## Running the game

Open `index.html` directly in a browser, or serve the directory with any static server, for example `python3 -m http.server`. No install step is needed.

There are no tests, linters, or build commands configured. Verify changes by loading the page and playing it. Check the console for errors.

## Code structure inside `index.html`

The script is one IIFE with `'use strict'`. Its sections, roughly in file order:

- **DOM references** - `getElementById` lookups at the top of the IIFE.
- **Constants** - `STORAGE_*` keys, `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES` (per-stage sky palettes), `ENEMY_TYPES`, `BOSSES`.
- **Adaptive performance** - `trackPerf` moves a `quality` level (0-2) up or down from the rolling frame time.
- **Audio** - Web Audio helpers: `ensureAudio`, `beep`, `noiseBurst`, and the `EXPLODE_SFX` mapping.
- **Input** - `keys` map fed by `keydown` and `keyup`, plus touch joystick and fire/bomb buttons.
- **Game state and flow** - `resetGame`, `update(dt)`, `endGame`, `togglePause`, `loseLife`.
- **Spawning and combat** - `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`, `fireWeapon`, `useBomb`, and the `fire*` boss-pattern helpers used by `BOSS_PATTERNS`.
- **Rendering** - `draw()`, `drawPlane`, `drawBossShip`, `drawClouds`. Everything is drawn to a 2D canvas.
- **Main loop** - `loop(now)` calls `update` and `draw` each frame. `dt` is clamped to 50 ms.

Game state is held in module-level `let` variables (`player`, `bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`, `boss`) that `resetGame()` reinitializes.

## Persistence

Settings and the best score are stored in `localStorage` under these keys:

- `skyfighter.bestScore`
- `skyfighter.muted`
- `skyfighter.difficulty`
- `skyfighter.reducedMotion`

Any new persisted setting should use the `skyfighter.*` prefix and be defined as a `STORAGE_*` constant. Reads and writes should tolerate a blocked or unavailable `localStorage`.

## Controls

- Move: arrow keys or WASD. On touch devices, use the on-screen joystick.
- Fire: Space (hold), or the fire button on touch devices.
- Bomb: X or Shift, or the bomb button.
- Pause: P or Escape.

## Conventions

- Keep the game in `index.html` unless the user asks for a split. Do not add a build system, framework, or dependencies without a clear reason.
- Match the existing style: 2-space indentation, single quotes in JS, `const`/`let`, arrow functions for callbacks, and short section banners like `// ---------- Name ----------`.
- Keep comments sparse and only where the logic is not obvious.
- Gameplay numbers (spawn rates, HP, speeds, scores) live in the constant tables near the top of the script. Prefer tuning them there over hard-coding values in functions.
- Preserve accessibility hooks: the `srLive` announcer, the reduced-motion toggle, and `prefers-reduced-motion` detection.
- Keep the layout working on phones. The page uses `touch-action: none`, safe-area insets, and a `resize()` that scales the canvas to fit.

## Git

- Develop on the branch the session names (currently `claude/gifted-tesla-z16k6b`). Do not push to other branches without permission.
- Commit with clear, descriptive messages. Do not open a pull request unless the user asks for one.
