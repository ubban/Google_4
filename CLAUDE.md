# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a small static web project. It contains a single-file browser game, **Sky Fighter**, a Raiden-style vertical shoot-'em-up.

| File | Purpose |
| --- | --- |
| `index.html` | The entire game: inline CSS, inline JavaScript, and a `<canvas>` renderer. No build step, no dependencies. |
| `README.md` | Project title only. |

There is no package manager, test suite, linter, or build tooling. Git history: `Initial commit`, then `Add Sky Fighter: Raiden-style vertical shoot-em-up game`.

## Running and verifying

- Open `index.html` directly in a browser, or serve the directory with any static server (for example `python3 -m http.server`).
- There are no automated tests. Verify changes by playing the game in a browser. For headless checks, Playwright is available in this environment (see the environment notes; do not reinstall it).
- Check the browser console for errors after any change to the JavaScript.

## Code structure of `index.html`

The file is organized into sections in roughly this order:

1. **`<head>`**: meta tags (viewport is locked, `user-scalable=no`), an inline SVG favicon, and the `<style>` block.
2. **Styles**: dark color scheme, full-viewport fixed layout, `touch-action: none` to block page gestures during play. The HUD is absolutely positioned over the canvas.
3. **Markup**: the canvas, HUD elements, touch controls (joystick zone and fire button), and overlay menus.
4. **Script** (single inline `<script>`):
   - **Persistence keys**: `STORAGE_BEST`, `STORAGE_MUTE`, `STORAGE_DIFF`, `STORAGE_REDUCED`. All are prefixed `skyfighter.`.
   - **Tuning tables**: `DIFFICULTIES` (spawn rates, enemy speed and fire chance, lives, bombs, boss HP multiplier), `KILLS_PER_STAGE`, `THEMES`, `ENEMY_TYPES`, `BOSSES`.
   - **Audio**: lazily created `AudioContext` via `ensureAudio()`, with `beep()` and `noiseBurst()` synthesis helpers.
   - **Input**: keyboard state map (`keys`, keyed by `KeyboardEvent.code`), plus touch joystick and fire button handlers.
   - **Game state and logic**: `resetGame()`, `update(dt)`, spawning (`spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`), weapons (`fireWeapon`), bombs (`useBomb`), scoring and combo chain (`addScore`, `breakChain`), and lives (`loseLife`, `endGame`).
   - **Boss patterns**: `BOSS_PATTERNS` and the `fire*` helpers (radial, aimed spread, columns, spiral, cross, homing, double ring). `updateBoss` and `killBoss` drive them.
   - **Rendering**: `draw()` and its helpers (`drawPlane`, `drawBossShip`, `drawClouds`). Fixed logical resolution `BASE_W` x `BASE_H` (480 x 720), scaled by `resize()`.
   - **Main loop**: `loop(now)` drives `update` and `draw` via `requestAnimationFrame`.

## Controls (keep these in sync with the code and the on-screen help)

- Move: arrow keys or WASD (read in `update`), or the touch joystick.
- Fire: `Space` (held), or the on-screen fire button / mouse button on touch and desktop.
- Bomb: `X` or `Shift`.
- Pause: `P` or `Escape`.
- Reduced motion: toggled in-game and persisted in `skyfighter.reducedMotion`; defaults to the OS `prefers-reduced-motion` setting when unset.
- `Space` is intercepted with `preventDefault()` to stop page scrolling.

## Conventions

- Keep the game in a single self-contained `index.html` unless the user asks for a restructure. Do not introduce a build step or external dependencies without a clear reason.
- Put tunable numbers in the existing constant tables (`DIFFICULTIES`, `BOSSES`, `ENEMY_TYPES`, and so on) rather than hard-coding them in logic.
- Persist settings only through the `skyfighter.*` localStorage keys. Wrap storage access so the game still runs if storage is unavailable (private windows, blocked site data).
- Respect the reduced-motion setting (`reducedMotion`) when adding screen shake, flashes, or other motion effects.
- Preserve mobile support: the layout must work at phone width, and touch controls must keep `touch-action: none` and `passive: false` where `preventDefault()` is called.
- Use `requestAnimationFrame` timing with the existing `dt` handling; do not add separate timers for game logic.
- Match the existing style: two-space indentation, single quotes in JavaScript, `const`/`let`, and short helper functions grouped under the section comments.

## Git

- Development branch for this repository: `claude/gifted-tesla-1kcq89`. Commit and push there; do not push to other branches without permission.
- Do not open a pull request unless the user asks for one.
