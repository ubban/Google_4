# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Google_4 is a small, dependency-free browser game, **Sky Fighter**: a Raiden-style vertical shoot-'em-up. The entire game lives in one file, `index.html`, which contains the markup, CSS (`<style>`, roughly lines 10–359), and JavaScript (`<script>`, roughly lines 422–1667).

Files:

- `index.html` — the whole game (no build step, no bundler, no external dependencies).
- `README.md` — a one-line placeholder.

There is no package manager, test suite, linter, or build configuration. Open `index.html` directly in a browser to run the game.

## Running and verifying

- Run: open `index.html` in a browser, or serve the directory with any static server (for example `python3 -m http.server`).
- There are no automated tests or lint commands. Verify changes by loading the page and playing: start a run, fire, use a bomb, pause, lose a life, reach a boss.
- Check at least one mobile-sized viewport when touching layout or touch controls, since the page targets phones (`touch-action: none`, joystick and fire/bomb buttons).

## Code layout (inside `index.html`)

The script is one `(function () { 'use strict'; ... })();` block with `const`/`let` state and function declarations. Major sections, in order:

1. **DOM references** — `getElementById` lookups for HUD, overlay, touch controls, and buttons (around line 426).
2. **Storage keys and helpers** — `STORAGE_*` constants for best score, mute, difficulty, and reduced motion, plus `announce()` for screen readers.
3. **Tuning data** — `DIFFICULTIES` (spawn rates, enemy speed, shoot chance, lives, bombs, boss HP multiplier), `KILLS_PER_STAGE`, `THEMES` (per-stage sky palettes), `ENEMY_TYPES` (grunt, interceptor, tank), and `BOSSES` (six boss designs, cycled by stage).
4. **Audio** — a lazily created `AudioContext` (`ensureAudio`), with `beep`, `noiseBurst`, and a `sfx` object of named sound effects.
5. **Input** — keyboard (`keys` map), touch joystick and buttons, and mouse targeting on the canvas.
6. **Game state** — `player`, `bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`, `boss`, plus score, lives, bombs, chain/multiplier, and stage counters. `resetGame()` initializes them.
7. **Spawning and entities** — `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`, and the per-frame update logic.
8. **Boss attacks** — `fireSpiralBurst`, `fireCross`, `fireHomingSeeded`, `fireDoubleRing`, driven by `updateBoss`. `killBoss` advances the stage.
9. **Game flow** — `loseLife`, `endGame`, `togglePause`.
10. **Rendering** — `draw()` and helpers (`drawPlane`, `drawBossShip`, `drawClouds`), using the 2D canvas context.
11. **Main loop** — `loop(now)` runs via `requestAnimationFrame`, with `trackPerf` adjusting the `quality` level (0 full, 1 medium, 2 low) when frames run slow.

Base logical resolution is 480×720 (`BASE_W`, `BASE_H`); `resize()` scales the canvas to fit the window up to 1.6×.

## Controls

- Move: arrow keys or WASD, or the touch joystick. The mouse also steers the ship when hovering the canvas.
- Fire: hold `Space`, or the on-screen fire button (`firing` state).
- Bomb: `X` or `Shift` (also the bomb button).
- Pause: `P` or `Escape`.

## Conventions

- **Keep it one file.** New game code goes in the existing `<script>` block and new styles in the existing `<style>` block unless the user asks for a split. Do not introduce a build step, framework, or package manager without a request.
- **No external assets or CDNs.** Graphics are drawn on canvas and sounds are synthesized with Web Audio. Icons are inline SVG data URIs.
- **Match the surrounding style.** The code uses 2-space indentation, single quotes, `const`/`let`, arrow functions for callbacks, and short comments only where a value's meaning isn't obvious.
- **Tuning values are data.** Prefer adding or adjusting entries in `DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, and `THEMES` over hard-coding numbers in logic.
- **Persist only through the existing keys.** Use the `STORAGE_*` constants for `localStorage`, and wrap access in a way that still works if storage is unavailable.
- **Accessibility.** Respect the reduced-motion setting (`reducedMotion`) when adding effects that shake or flash the screen, and route status messages through `announce()`.
- **Mobile first.** Keep the layout working at phone widths and don't rely on hover-only interactions.

## Git workflow

- Develop on the branch specified by the task (currently `claude/gifted-tesla-knepw0`); don't push to `main` without explicit permission.
- Commit with clear, descriptive messages and push with `git push -u origin <branch>`.
- Only open a pull request when the user explicitly asks for one.
