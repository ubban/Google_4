# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo contains **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs in the browser. The whole game is a single self-contained file:

- `index.html` (~1,670 lines): inline `<style>`, the HTML/UI markup, and one inline `<script>` wrapped in an IIFE (`'use strict'`).
- `README.md`: a placeholder. It has no build or run instructions.

There is no `package.json`, bundler, framework, test suite, linter, or CI config. Do not invent npm scripts or test commands. To run the game, open `index.html` in a browser (or serve the directory with any static server).

## Working on the codebase

- Keep the game as a single file unless the user asks for a split. Do not add a build step or dependencies without a clear request.
- Make changes in `index.html`. Edit only the section you need. Do not reformat or reorder unrelated code.
- Verify changes by running the game in a browser and playing through the affected path. There are no automated tests, so say so explicitly when reporting results rather than claiming the code is tested.
- Use `git` on the designated feature branch. Do not push to other branches without permission.

## Code map (`index.html`)

Approximate line locations; they will drift as the file changes, so search for the names rather than trusting the numbers.

| Area | Where to look |
| --- | --- |
| Persistence keys (`skyfighter.*`) | Top of the script: `STORAGE_BEST`, `STORAGE_MUTE`, `STORAGE_DIFF`, `STORAGE_REDUCED` |
| Difficulty presets | `DIFFICULTIES` (spawn rates, enemy speed, shot chance, lives, bombs, boss HP multiplier) |
| Stage themes (sky gradients, names) | `THEMES` |
| Enemy and boss definitions | `ENEMY_TYPES` (grunt, interceptor, tank); `BOSSES` (six bosses, cycled by stage) |
| Boss attack patterns | `BOSS_PATTERNS` and the `fire*` helpers (radial, spread, columns, spiral, cross, homing, double ring) |
| Game state reset | `resetGame()` |
| Per-frame simulation | `update(dt)`; `updateBoss(dt)`; `applyPowerUp(type)` |
| Rendering | `draw()`; `drawPlane()`; `drawBossShip()`; `drawClouds()` |
| Game loop | `loop(now)`; `dt` is clamped to 0.05 s |
| Audio | `ensureAudio()`, `beep()`, `noiseBurst()`, and the `sfx` object (Web Audio, no asset files) |
| Input | Keyboard handlers on `window`; pointer handling in `canvasPointFromEvent`; touch joystick and buttons (`joystick*`, `fireBtn`, `bombBtn`) |
| Adaptive quality | `trackPerf(dt)` lowers or raises `quality` based on the rolling average frame time |
| Accessibility | `announce()` writes to the `srLive` live region; reduced-motion toggle via `motionBtn` and `screenShake` |

## Gameplay and controls

- Move: Arrow keys or WASD, or the on-screen joystick on touch devices.
- Fire: Space (hold), or the fire button on touch devices.
- Bomb: `X` or Shift, or the bomb button. Bombs are limited per difficulty.
- Pause: `P` or Escape.
- Difficulty (`easy` / `normal` / `hard`) is chosen from the overlay and saved to `localStorage`.
- Every `KILLS_PER_STAGE` (18) kills in a stage triggers that stage's boss. Defeating the boss advances to the next stage. Stages cycle through `THEMES`, and bosses cycle through `BOSSES` by stage number.

## Conventions

- Game constants live in the config objects near the top of the script (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `THEMES`). Prefer adding or tuning values there over hard-coding numbers in logic.
- Read and write `localStorage` only through the existing `STORAGE_*` keys. Wrap new storage access in `try/catch`, since storage can be blocked or unavailable.
- Any new user-facing status change should also go through `announce()` so screen-reader users get it.
- Respect `reducedMotion` for new screen shake, flash, or particle effects.
- Keep audio generated with Web Audio. Do not add audio files.
- Audio starts only after a user gesture (`ensureAudio()`); keep it that way to satisfy browser autoplay rules.

## Git

- Develop on the branch specified by the session. Commit with clear messages and push with `git push -u origin <branch>`.
- Do not open a pull request unless the user asks for one.
