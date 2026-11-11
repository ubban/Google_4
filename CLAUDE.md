# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo is **Sky Fighter**, a single-file, browser-based, Raiden-style vertical shoot-'em-up. There is no build step, package manager, test suite, or server-side code.

- `index.html` — the entire game: inline `<style>` (~lines 10–360), markup (~lines 361–420), and one IIFE `<script>` (~lines 422–1667) that holds all game logic, rendering, audio, and input.
- `README.md` — a placeholder with no real project documentation.
- Git history: `main` holds the initial commit; the working branch is `claude/gifted-tesla-1dqqtj`.

## Running and verifying

- Open `index.html` directly in a browser, or serve the directory with any static server (e.g. `python3 -m http.server`).
- There are no tests, linters, or build commands. Verify changes by playing the game in a browser (desktop keyboard and, if possible, touch emulation).
- Keep the file a single self-contained HTML document. Do not introduce external CDN scripts, fonts, or assets unless the user asks.

## Code layout inside `index.html`

The script is wrapped in one IIFE with `'use strict'`-style module-private state. Key sections, in source order:

1. **DOM references** — `getElementById` lookups for HUD, overlay, touch controls, and boss bar.
2. **Storage keys** — `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion` (all in `localStorage`, all read/written defensively as strings).
3. **Tuning tables** — `DIFFICULTIES` (easy/normal/hard: spawn rate, enemy speed, shoot chance, lives, bombs, boss HP multiplier), `KILLS_PER_STAGE = 18`, `THEMES` (six sky palettes, cycled by stage), `ENEMY_TYPES` (grunt, interceptor, tank), `BOSSES` (six designs, cycled by stage).
4. **Audio** — a lazily created `AudioContext` (`ensureAudio`), with `beep`, `noiseBurst`, and a named `sfx` table. Audio is unlocked on first user input.
5. **Adaptive quality** — `trackPerf` samples frame times over 45 frames and steps a `quality` level (0–2) up or down to hold ~40–56 FPS.
6. **Input** — keyboard (arrows/WASD move, Space fires, X/Shift bomb, P/Esc pause), mouse/pointer steering, and touch (virtual joystick plus FIRE/BOMB buttons, shown only on touch devices).
7. **Game state** — module-level `let` variables (`player`, `bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`, `boss`, `score`, `lives`, `bombCount`, `stage`, `chain`/`multiplier`, etc.). `resetGame()` reinitializes them.
8. **Drawing** — canvas functions `drawPlane`, `drawBossShip`, `drawClouds`, and `draw()`.
9. **Simulation** — `update(dt)`, `updateBoss(dt)`, `spawnEnemy()`, collision and power-up handling (`applyPowerUp`: life, bomb, and weapon types).
10. **Loop** — `loop(now)` clamps `dt` to 50 ms, calls `update`/`draw` only while running and unpaused, then `requestAnimationFrame`.

Game flow: the overlay start screen sets difficulty → `resetGame()` → enemies spawn → after `KILLS_PER_STAGE` kills a boss is warned and spawned → defeating it advances `stage`, which selects the next theme and boss (both indexed by `(stage - 1) % length`).

## Conventions

- Canvas logical size is `BASE_W × BASE_H` (480 × 720), scaled to fit the window by `resize()`. Write game coordinates in the logical space, never in CSS pixels.
- Use `dt` (seconds) for all time-based motion so behavior is frame-rate independent.
- Respect `reducedMotion`: screen shake, flashes, and pulsing should be gated on it (see `screenShake`, `flashHit`, `flashBomb`).
- Announce important state changes through the `announce()` live region for screen readers, and keep `aria-*` and `title` attributes on controls current.
- Store only the four `skyfighter.*` keys in `localStorage`, and wrap access so the game still runs when storage is blocked.
- Keep styling and logic in the existing sections; add new tuning values to the existing constant tables rather than hard-coding them in the loop.
- Preserve the mobile layout (safe-area insets, `touch-action: none`, no page scrolling).

## Git workflow

- Develop on `claude/gifted-tesla-1dqqtj` and push with `git push -u origin claude/gifted-tesla-1dqqtj`.
- Do not open a pull request unless the user asks for one.
- Commit messages should be short, imperative, and describe the gameplay or file change.
