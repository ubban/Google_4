# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo contains **Sky Fighter**, a single-page HTML5 canvas vertical shoot-em-up (Raiden-style). The whole game lives in one file, `index.html`: markup (~lines 1–362), CSS (`<style>`, ~line 10), canvas element (~line 363), and one inline `<script>` IIFE (~line 422 to end). There is no build step, no package manager, no bundler, and no test suite.

`README.md` is a placeholder and contains only the title.

## Running and testing

- Open `index.html` directly in a browser, or serve the directory with any static server (e.g. `python3 -m http.server`).
- There are no automated tests or linters configured. Verify changes by playing the game in a browser, ideally at both desktop and mobile (touch) viewport sizes.
- Keep the file dependency-free: no external scripts, fonts, or CDNs are currently used. Don't add any without a clear reason.

## Code layout inside `index.html`

The game script is one IIFE with `'use strict'`. Major sections, in source order:

- **DOM references**: `const` lookups for every HUD element, overlay, and touch control.
- **Storage keys**: `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion` (all `localStorage`).
- **Adaptive performance**: `trackPerf` samples frame time and steps a `quality` level (0–2) up or down.
- **Tuning tables**:
  - `DIFFICULTIES` (easy / normal / hard): spawn rates, enemy speed, shoot chance, starting lives and bombs, boss HP multiplier.
  - `KILLS_PER_STAGE` (18): kills needed to advance a stage.
  - `THEMES`: six sky palettes cycled per stage.
  - `ENEMY_TYPES`: `grunt`, `interceptor` (weaving), `tank` (4 HP, unlocks at stage 2).
  - `BOSSES`: six boss designs, chosen by `(stage - 1) % BOSSES.length`.
- **Audio**: a lazily created `AudioContext` (`ensureAudio`) with synthesized `beep` and `noiseBurst` effects. Nothing is loaded from files.
- **Input**: keyboard (`keys` map on `e.code`), touch joystick and fire/bomb buttons, and mouse on the canvas.
- **Sizing**: `BASE_W = 480`, `BASE_H = 720` logical resolution, scaled by `resize()`.
- **Game state**: `resetGame()` initializes `player`, `bullets`, `enemies`, `enemyBullets`, `particles`, power-ups, and boss state.
- **Simulation**: `update(dt)` drives spawning, collisions, weapons (`fireWeapon`), bombs (`useBomb`), score/combo (`addScore`, `breakChain`), and power-ups (`applyPowerUp`).
- **Bosses**: `spawnBoss`, `BOSS_PATTERNS` (radial, aimed spread, columns, spiral, cross, homing, double ring), `updateBoss`, `killBoss`.
- **Rendering**: `draw()` plus helpers `drawPlane`, `drawBossShip`, `drawClouds`. Screen shake and hit flashes are applied here.
- **Main loop**: `loop(now)` clamps `dt` to 0.05 s, calls `update` and `draw` only when running and not paused, and re-schedules via `requestAnimationFrame`.

## Controls

| Action | Keyboard | Touch |
|---|---|---|
| Move | Arrow keys / WASD | Left-side joystick |
| Fire | Space (hold) | Fire button (hold) |
| Bomb | X or Shift | Bomb button |
| Pause | P or Escape | HUD pause button |
| Mouse | Move pointer over canvas to steer; hold button on canvas to fire | — |

Power-ups include weapon level, bomb, and extra life. Lives are shown as hearts; bombs are shown in the HUD.

## Conventions

- **Single file**: keep game logic, styles, and markup in `index.html` unless the user asks for a split.
- **Style**: match the existing code — 2-space indent, single quotes, `const`/`let`, arrow functions for callbacks, and small named helper functions in the IIFE.
- **Tuning changes**: prefer editing the tables at the top of the script (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `THEMES`) over hard-coding numbers in logic.
- **Persistence**: any new saved setting should use a `skyfighter.*` key and be wrapped so the game still works if `localStorage` is unavailable or throws.
- **Accessibility**: respect `prefers-reduced-motion` (the `reducedMotion` flag disables screen shake and similar effects) and keep the `srLive` announcer updated for important state changes.
- **Performance**: avoid allocations in the per-frame path where practical; the adaptive `quality` level exists to shed effects on slow devices.

## Git workflow

- Feature work for this repo is developed on `claude/gifted-tesla-ush5pn`. Commit there and push with `git push -u origin <branch>`.
- Use clear, descriptive commit messages that explain the gameplay or code change.
- Do not open pull requests unless explicitly asked.
