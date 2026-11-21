# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo contains **Sky Fighter**, a single-file, browser-based vertical shoot-'em-up in the style of Raiden. There is no build step, no package manager, and no test suite.

- `index.html` — the entire game: inline CSS (`<style>`), markup (HUD, overlays, touch controls, `<canvas id="gameCanvas">`), and one inline `<script>` IIFE that holds all game logic, rendering, audio, and input.
- `README.md` — minimal placeholder.

## Running and testing

- Open `index.html` directly in a browser, or serve the directory with any static server (e.g. `python3 -m http.server`).
- There are no automated tests, linters, or build commands. Verify changes by playing the game in a browser, including the touch controls when relevant (test at phone width as well as desktop).

## Code layout inside `index.html`

The script is wrapped in `(function () { 'use strict'; ... })();`. Keep new code inside that wrapper; do not add globals.

Major sections, roughly in file order:

1. **DOM references** — `const xxxEl = document.getElementById(...)` at the top of the script.
2. **Persistence keys** — `STORAGE_BEST`, `STORAGE_MUTE`, `STORAGE_DIFF`, `STORAGE_REDUCED` (`skyfighter.*` in `localStorage`).
3. **Tuning data** — `DIFFICULTIES` (spawn rates, enemy speed, lives, bombs, boss HP multiplier), `KILLS_PER_STAGE`, `THEMES` (per-stage sky/cloud/sun palettes), `ENEMY_TYPES`, `BOSSES`.
4. **Audio** — `ensureAudio`, `beep`, `noiseBurst`, `panFor`, `EXPLODE_SFX`. Audio is created lazily on the first user gesture.
5. **Input** — keyboard handlers on `window`, a virtual joystick and fire/bomb buttons for touch, and `canvasPointFromEvent` for pointer coordinates.
6. **Game state and lifecycle** — `resetGame`, `loseLife`, `endGame`, `togglePause`, stage progression, `addScore` / combo chain (`breakChain`).
7. **Entities and spawning** — `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`, `spawnExplosion`, `rectsOverlap` for collision.
8. **Weapons and bombs** — `fireWeapon`, `useBomb`, `applyPowerUp`.
9. **Bosses** — `BOSS_PATTERNS` (bullet patterns such as radial, aimed spread, spiral, cross, homing) with `updateBoss` / `killBoss`.
10. **Update and draw** — `update(dt)` advances the simulation; `draw()` renders everything; `drawPlane`, `drawBossShip`, `drawClouds`.
11. **Main loop** — `loop(now)` calls `trackPerf`, then `update` and `draw` while `running && !paused`, using a `dt` clamped to 0.05 s.
12. **Layout** — `BASE_W = 480`, `BASE_H = 720` define the logical resolution; `resize()` scales the canvas to fit the viewport.

## Controls (for reference when changing input)

| Action | Keyboard | Touch |
| --- | --- | --- |
| Move | Arrow keys or WASD | Left-side joystick |
| Fire | Space (hold) | Fire button |
| Smart bomb | X or Shift | Bomb button |
| Pause | P or Esc | Pause button |
| Mute | Mute button | Mute button |

## Conventions

- Keep the game in a single self-contained `index.html` unless the user asks for a split. Do not introduce a build system, framework, or external dependencies without asking.
- Use `dt`-scaled movement in `update()` so speed is frame-rate independent.
- Keep game-tuning numbers in the data objects (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `THEMES`) rather than hard-coding them in logic.
- Preserve accessibility hooks: the `srLive` live region is used for announcements via `announce()`, and the reduced-motion setting gates screen shake and similar effects.
- Persisted values go through the `skyfighter.*` `localStorage` keys. Wrap storage access so the game still works when storage is unavailable (private mode, blocked site data).
- Keep the layout working at phone width with no horizontal page scroll; the page uses `overflow: hidden` and `touch-action: none` by design.
- Match the existing code style: 2-space indentation, single quotes in JS, `const` by default, and arrow functions for event handlers.

## Git

- Development branch for this work: `claude/gifted-tesla-ixasqs`. Do not push to other branches without explicit permission.
- Do not open a pull request unless the user asks for one.
