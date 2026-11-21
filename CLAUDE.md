# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo contains **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs entirely in the browser.

- The whole game is a single self-contained file: `index.html` (~1,670 lines). It holds the HTML markup, the inline `<style>` block, and one inline `<script>` that is wrapped in an IIFE (`(function () { 'use strict'; ... })();`).
- `README.md` is a placeholder and has no project documentation.
- There is **no build step, no package manager, no bundler, no test suite, and no linter** configured. Do not invent `npm` / `make` commands.

## Running and verifying changes

- Open `index.html` directly in a browser, or serve the directory with any static server (for example `python3 -m http.server` from the repo root) and visit it.
- There are no automated tests. Verify gameplay changes by playing in a browser, and check the browser console for errors. For headless checks, Playwright with the pre-installed Chromium works.
- Keep the file loadable as-is: the game must keep working when opened from `file://`, so do not add external script or asset dependencies without a clear reason.

## Code structure (inside the `<script>` block)

The script is organized roughly as follows. Keep new code in the same style and location.

1. **DOM references and constants**: element lookups (`const xxxEl = document.getElementById(...)`), `STORAGE_*` keys, `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES`, `ENEMY_TYPES`, `BOSSES`.
2. **Persistence and settings**: best score, mute, difficulty, and reduced-motion preference are stored in `localStorage` under the `skyfighter.*` keys (`skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion`).
3. **Audio**: a Web Audio API synth (`ensureAudio`, `beep`, `noiseBurst`, `panFor`). Sound effects are generated procedurally; there are no audio files. `EXPLODE_SFX` maps enemy types to explosion sounds.
4. **Rendering helpers**: `renderHearts`, `renderBombs`, HUD flashes, combo popups, stage banners, screen shake, and `resize`. The logical play field is `BASE_W = 480` by `BASE_H = 720`, scaled to fit the canvas.
5. **Input**: keyboard (`keydown` / `keyup` populate the `keys` map by `e.code`), pointer, and on-screen touch controls (joystick, fire, bomb).
6. **Game state and logic**: `resetGame`, `update(dt)`, `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`, `applyPowerUp`, `fireWeapon`, `useBomb`, `loseLife`, `endGame`, `togglePause`.
7. **Boss patterns**: `BOSS_PATTERNS` is an array of bullet-pattern functions (radial, aimed spread, columns, spiral, cross, homing, double ring). `updateBoss` and `killBoss` drive them.
8. **Drawing**: `draw()` plus sprite helpers (`drawPlane`, `drawBossShip`, `drawClouds`).
9. **Main loop**: `loop(now)` runs on `requestAnimationFrame`. It computes `dt`, calls `update`, then `draw`. A `trackPerf` helper also exists.

## Gameplay reference

- **Controls**: move with Arrow keys or WASD; fire with Space (or hold the fire button on touch); `X` / Shift to use a smart bomb; `P` or Escape to pause. Touch layout has a joystick, fire, and bomb buttons.
- **Difficulty** (`DIFFICULTIES`): `easy`, `normal` (default), `hard`. Each sets spawn rate, enemy speed, enemy shoot chance, starting lives, starting bombs, and a boss HP multiplier.
- **Stages**: after `KILLS_PER_STAGE` (18) kills, a boss spawns (`spawnBoss`). Defeating it advances to the next stage. Stage N uses `THEMES[(N-1) % 6]` and `BOSSES[(N-1) % 6]`, so both cycle. Each boss has its own `hpMul` and attack pattern.
- **Enemies** (`ENEMY_TYPES`): `grunt` (stage 1+), `interceptor` (stage 1+, weaving), `tank` (stage 2+, 4 HP).

## Conventions

- Keep the game self-contained in `index.html`. Do not split it into modules or add a build step unless the user asks for that.
- Use plain ES2015+ JavaScript that runs in current evergreen browsers. Match the existing style: 2-space indentation, single quotes, `const` / `let`, and section comments.
- Balance changes belong in the constant tables (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `BOSS_PATTERNS`) rather than being hard-coded inside functions.
- Respect the reduced-motion setting: visual effects such as screen shake should check the `reducedMotion` flag.
- Keep touch and keyboard input working together. Any new control needs both paths, or a clear reason why only one applies.
- Do not store anything in `localStorage` beyond the existing `skyfighter.*` keys without updating this file.

## Git

- Development branch for this work: `claude/gifted-tesla-en8mx8`. Commit and push there; do not push to other branches without explicit permission.
- Do not open a pull request unless the user asks for one.
