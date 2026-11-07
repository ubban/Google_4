# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo is **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs in the browser. The entire game is a single self-contained file:

- `index.html` (~1,670 lines): inline `<style>`, markup (HUD, overlay, touch controls, pause/mute/motion buttons), and one inline `<script>` wrapped in an IIFE with `'use strict'`.
- `README.md`: a minimal placeholder. It has no build or run instructions.

There is no build step, package manager, test suite, linter, or server-side code. Do not add a framework, bundler, or dependencies unless asked.

## Running and verifying

- Open `index.html` directly in a browser, or serve the directory with any static server (for example `python3 -m http.server`).
- There are no automated tests. Verify changes by playing the game in a browser: start a run, exercise the changed behavior, and check the console for errors.
- For headless checks, Playwright/Chromium is available in the cloud environment. Load `file:///…/index.html` and watch for `pageerror` / console errors.

## Code structure (inside the `<script>`)

The script is organized by section comments. Key pieces, roughly in file order:

- **DOM refs and constants**: element lookups at the top; `STORAGE_*` keys (`skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion`).
- **Adaptive performance** (`trackPerf`): a 0/1/2 `quality` level, adjusted from the average frame time over 45 frames.
- **Data tables**: `DIFFICULTIES` (easy/normal/hard: spawn rates, enemy speed, shot chance, lives, bombs, boss HP multiplier), `THEMES` (six sky palettes, one per stage), `ENEMY_TYPES` (grunt, interceptor, tank), `BOSSES` (six bosses, cycling by stage), `KILLS_PER_STAGE` (18).
- **Audio** (`ensureAudio`, `beep`, `noiseBurst`): Web Audio synthesis only. There are no audio files.
- **Input**: keyboard (`keys` map keyed by `e.code`), mouse (steer and click to fire), and touch (joystick on the left, FIRE and BOMB buttons). The touch controls are enabled only on touch devices.
- **Game state and logic**: `resetGame`, `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`, `fireWeapon`, `useBomb`, `update(dt)`, `applyPowerUp`, `updateBoss`, `loseLife`, `endGame`, `togglePause`.
- **Boss patterns**: `BOSS_PATTERNS` plus helpers (`fireRadial`, `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`, `fireCross`, `fireHomingSeeded`, `fireDoubleRing`).
- **Rendering**: `draw()` plus helpers (`drawPlane`, `drawBossShip`, `drawClouds`). Rendering uses Canvas 2D on `#gameCanvas`. `resize()` fits a fixed `480×720` logical playfield (`BASE_W`, `BASE_H`) into the viewport.
- **Main loop**: `loop(now)` clamps `dt` to 50 ms, runs `update` and `draw` while running and not paused, and calls `requestAnimationFrame`.

Game-over and personal-best data are written to `localStorage`. Read and write it defensively, because storage can be unavailable (for example in private windows).

## Controls (as implemented)

| Action | Input |
| --- | --- |
| Move | Mouse, Arrow keys, or WASD; touch joystick on mobile |
| Fire | Hold Space, or hold mouse/FIRE button |
| Bomb | X or Shift; BOMB button on touch devices |
| Pause | P or Esc |
| Mute / reduced motion | On-screen buttons (`#muteBtn`, `#motionBtn`) |

Weapon power-ups toggle between `vulcan` and `laser`. Life and bomb pickups are also dropped in play.

## Conventions

- **Keep it one file.** Put new CSS in the existing `<style>` block and new JS inside the existing IIFE, following the section-comment layout.
- **Plain JS and Canvas 2D.** Match the existing style: `const`/`let`, arrow functions for callbacks, no transpilation, no external libraries or CDNs.
- **Tune through the data tables.** Enemy, boss, difficulty, and theme values live in the constant tables above. Prefer editing those over hardcoding numbers into logic.
- **Respect the accessibility hooks.** Keep the reduced-motion path working (screen shake, flashes, and pulses are gated by `reducedMotion`). Keep the `#srLive` announcements for stage changes and similar events.
- **Preserve touch support.** Any new control needs a touch path as well as keyboard/mouse, and the layout must stay usable at phone width with `touch-action: none` on the game.
- **Use `Math.min(dt, 0.05)`-style timing.** Game logic is frame-rate independent through `dt`; do not introduce per-frame fixed increments.

## Git

- Development branch for this workspace: `claude/gifted-tesla-jt8xer`. Push with `git push -u origin <branch>`.
- Do not open a pull request unless explicitly asked.
- Write clear, descriptive commit messages in the imperative mood.
