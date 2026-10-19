# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Google_4 is a small static web project. Its only real content is **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs in the browser on a `<canvas>`.

## Repository layout

- `index.html` — the whole game: inline `<style>` (about lines 10–361), the markup (about lines 362–421), and one inline `<script>` (about lines 422–1668) wrapped in an IIFE with `'use strict'`.
- `README.md` — a placeholder with no project documentation.
- No `package.json`, build step, bundler, linter, or test suite. Nothing is installed or compiled.

## Running and verifying

- Open `index.html` directly in a browser, or serve the directory with any static server (for example `python3 -m http.server`).
- There are no automated tests. Verify changes by playing the game in a browser. Check the start flow, pause, difficulty switching, bombs, boss fights, game over, and the mobile touch controls.
- Keep the page free of external dependencies. It currently loads nothing from a CDN, and it should stay that way unless there is a clear reason to change it.

## Code structure inside `index.html`

The script is organized into sections marked by `// ---------- Name ----------` comments. Keep new code in the matching section.

- **Config tables:** `DIFFICULTIES` (spawn rates, enemy speed, shot chance, lives, bombs, boss HP multiplier), `KILLS_PER_STAGE`, `THEMES` (per-stage sky palettes), `ENEMY_TYPES` (grunt, interceptor, tank), and `BOSSES` (six bosses with colors and HP multipliers).
- **Adaptive performance:** `trackPerf` watches frame times and sets `quality` (0 full, 1 medium, 2 low).
- **Audio:** Web Audio helpers `beep` and `noiseBurst`, created lazily through `ensureAudio()`. Sound is muted with the mute button.
- **Input:** keyboard and mouse handlers on `window`, plus a touch joystick and FIRE/BOMB buttons that appear on touch devices.
- **Game state and logic:** `resetGame`, `update(dt)`, `spawnEnemy`, `spawnBoss`, `fireWeapon`, `useBomb`, `applyPowerUp`, `updateBoss`, `loseLife`, `endGame`, `togglePause`.
- **Boss attack patterns:** `BOSS_PATTERNS` plus the `fire*` helpers (radial, aimed spread, columns, spiral, cross, homing, double ring).
- **Rendering:** `drawPlane`, `drawBossShip`, `drawClouds`, and `draw()`. All rendering is immediate-mode canvas drawing. There are no image assets.
- **Main loop:** `loop(now)` caps `dt` at 0.05 s and runs update and draw only while the game is running and not paused.

## Persistence

The game stores four keys in `localStorage`. Keep the names unchanged, because they are how players keep their settings and best score:

- `skyfighter.bestScore`
- `skyfighter.muted`
- `skyfighter.difficulty`
- `skyfighter.reducedMotion`

The reduced-motion preference defaults to the OS `prefers-reduced-motion` setting until the player changes it.

## Conventions

- Match the existing style: 2-space indentation, single quotes, arrow functions for callbacks, `const` by default, and short comments only where they add information.
- Keep the file a single self-contained HTML document unless the user asks for a restructure. Splitting it into modules or adding a build step is a bigger change than most tasks call for.
- Use the `skyfighter.*` prefix for any new storage key.
- Gameplay numbers live in the config tables near the top of the script. Prefer changing those over hard-coding values in the update or draw code.
- Keep the UI usable at phone width and preserve the touch controls. The layout is responsive, with `resize()` scaling to a `480×720` base size.
- Respect the accessibility features already present: the `aria-live` announcer (`announce()`), the reduced-motion toggle, and the `sr-only` status region.

## Git

- Development happens on feature branches. Commit with clear messages and push to the branch named in the task. Do not push to other branches without permission.
- Do not open a pull request unless the user asks for one.
