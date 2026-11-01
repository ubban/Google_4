# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo is a small, dependency-free web game: **Sky Fighter**, a Raiden-style vertical shoot-'em-up.

- The entire game lives in a single file, `index.html` (~1,670 lines): inline `<style>`, the markup/HUD/overlay, and one inline `<script>` wrapped in an IIFE with `'use strict'`.
- It renders to a `<canvas id="gameCanvas">` using the 2D context. The logical play field is `BASE_W = 480` x `BASE_H = 720`, scaled by `resize()`.
- `README.md` is a placeholder. There is no build system, package manager, test suite, linter, or CI config.

## Running and testing

- No build step. Open `index.html` directly in a browser, or serve the directory with any static server (e.g. `python3 -m http.server`).
- There are no automated tests. Verify changes by playing the game in a browser, and check both desktop (keyboard) and touch layouts when touching input, HUD, or layout code.
- Do not add a build tool, framework, or package manager unless the user asks for one. Keep the project a single self-contained HTML file unless there is a clear reason to split it.

## Code map (`index.html`)

Within the main `<script>`, the code is organized roughly as follows (search by name to find it):

| Area | Key symbols |
| --- | --- |
| Persistence keys | `STORAGE_BEST`, `STORAGE_MUTE`, `STORAGE_DIFF`, `STORAGE_REDUCED` (all `skyfighter.*` in `localStorage`) |
| Tuning data | `DIFFICULTIES` (easy/normal/hard), `KILLS_PER_STAGE`, `THEMES` (per-stage sky palettes), `ENEMY_TYPES` (grunt, interceptor, tank), `BOSSES` (6 bosses) |
| Adaptive quality | `quality`, `trackPerf()` — drops visual detail when frame time is high |
| Audio | `ensureAudio()`, `beep()`, `noiseBurst()`, `EXPLODE_SFX` — Web Audio, created lazily on first user gesture |
| Input | `keys` map (keyboard), joystick (`joystickStart/Move/End`), fire/bomb touch buttons, mouse on canvas (`canvasPointFromEvent`) |
| State lifecycle | `resetGame()`, `endGame()`, `loseLife()`, `togglePause()` |
| Simulation | `update(dt)`, `spawnEnemy()`, `spawnBoss()`, `updateBoss(dt)`, `killBoss()`, `applyPowerUp()`, `useBomb()`, `fireWeapon()` |
| Boss attacks | `BOSS_PATTERNS` and the `fire*` helpers (radial, aimed spread, columns, spiral, cross, homing, double ring) |
| Rendering | `draw()`, `drawPlane()`, `drawBossShip()`, `drawClouds()` |
| Main loop | `loop(now)` — `dt` is clamped to 50 ms; `update` and `draw` run only when `running && !paused` |

### Controls

- Keyboard: arrows or WASD to move, Space (or hold) to fire, X or Shift to bomb, P or Escape to pause.
- Touch: on-screen joystick, fire button, bomb button.
- Mouse: move/fire on the canvas.

### Game rules worth knowing

- Difficulty changes lives, bombs, enemy spawn rate, enemy shoot chance, and boss HP multiplier.
- Stages advance every `KILLS_PER_STAGE` (18) kills; each stage ends in a boss fight with a health bar.
- Weapons (Vulcan / Laser) have 3 power levels. Power-ups are `life`, `bomb`, and `weapon`.
- Score uses a kill-chain multiplier (`addScore`, `breakChain`).

## Conventions

- Keep the vanilla JS style: `const`/`let`, plain functions, no transpilation, no external libraries or CDN loads.
- Keep constants for tuning in the top-level data objects (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `THEMES`) rather than hard-coding numbers in logic.
- Respect the existing accessibility features when changing UI or effects: the `reduced-motion` class / `reducedMotion` flag, `announce()` for screen-reader updates, and the `prefers-reduced-motion` default.
- Wrap all `localStorage` access in a way that tolerates it being unavailable or throwing, and keep the `skyfighter.*` key names stable so saved scores and settings survive.
- Preserve the `'use strict'` IIFE scope; do not leak new globals.
- Match the existing indentation (2 spaces) and the section comment style (`// ---------- Name ----------`).

## Git workflow

- Develop on the branch specified by the task (currently `claude/gifted-tesla-1k1ycs`). Do not push to other branches without explicit permission.
- Write clear commit messages describing the gameplay or code change.
- Do not open a pull request unless the user explicitly asks for one.
