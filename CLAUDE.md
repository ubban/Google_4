# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Sky Fighter is a Raiden-style vertical shoot-'em-up that runs in the browser. The entire game lives in a single self-contained file, `index.html` (inline `<style>` and one inline `<script>`, drawn on a `<canvas>` with the 2D context). There is no framework, no bundler, no package manager, and no external network dependencies.

`README.md` is only a placeholder title and timestamp; it does not describe the game.

## Repository layout

- `index.html` — the whole game: markup, CSS, and JavaScript.
- `README.md` — placeholder.

## Running and testing

- There is no build step, dependency install, test suite, or linter configured.
- To run the game, open `index.html` directly in a browser, or serve the directory with any static server (for example `python3 -m http.server`) and load it.
- Verify changes by playing in a browser. Check at least: starting a game, shooting, bombs, pause/resume, game over, and the best-score update.
- If you add tooling (tests, linting, a build), document the commands in this file.

## Code structure (inside `index.html`)

The script is one IIFE with `'use strict'`. Major sections, in source order:

- **DOM references and constants** — element lookups, `STORAGE_*` keys, `DIFFICULTIES`, `KILLS_PER_STAGE` (18), `THEMES` (per-stage sky palettes), `ENEMY_TYPES` (grunt, interceptor, tank), `BOSSES` (six bosses, cycled by stage).
- **Persistence** — `localStorage` keys `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion`. Reads are not wrapped in try/catch today; keep that in mind if you touch them.
- **Audio** — Web Audio synthesis (`beep`, `noiseBurst`, and the `sfx` object). Audio is created lazily by `ensureAudio()` after a user gesture.
- **Adaptive performance** — `trackPerf` samples frame time and steps a `quality` level (0 full, 1 medium, 2 low) up or down.
- **Input** — keyboard (`keydown`/`keyup` on `keys[e.code]`), mouse on the canvas, and touch controls (virtual joystick on the left, FIRE and BOMB buttons).
- **Game state and update** — `resetGame`, `update(dt)`, `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`, `applyPowerUp`, `fireWeapon`, `useBomb`, `loseLife`, `endGame`, `togglePause`.
- **Boss patterns** — `BOSS_PATTERNS` and the `fire*` helpers (radial, aimed spread, columns, spiral, cross, homing, double ring), driven by `updateBoss`.
- **Rendering** — `draw()`, `drawPlane`, `drawBossShip`, `drawClouds`.
- **Main loop** — `loop(now)` clamps `dt` to 50 ms, then calls `update` and `draw` while running and not paused.

## Game rules worth knowing

- Lives and bombs depend on difficulty: Easy 4 lives / 3 bombs, Normal and Hard 3 lives / 2 bombs. Hard also raises enemy speed and fire rate and gives bosses more HP.
- Power-ups drop from destroyed enemies: life (6%), bomb (8%), weapon (20%). Weapon power-ups raise weapon level up to 3, then toggle between VULCAN and LASER.
- Taking a hit grants 1.5 s of invulnerability and breaks the score combo chain.
- Stages advance every 18 kills, and bosses appear on stage transitions.

## Conventions

- Keep the single-file structure unless the user asks for a split. New code goes in the existing IIFE and follows its style: `const`/`let`, arrow callbacks, 2-space indentation, short section comments with `// ---------- Name ----------` headers.
- Match the surrounding naming: enemy and boss data live in the top-level constant tables; per-entity state uses plain object literals pushed onto arrays.
- Keep the game playable without a network connection and without any dependency installed.
- Preserve the accessibility hooks: `announce()` for screen-reader status, `aria-live` region `#srLive`, and the `reduced-motion` body class that disables shake, flash, and pulse animations.
- Preserve mobile support: the viewport meta, `touch-action: none`, and the touch control layout must keep working at phone width.
- Do not add analytics, remote calls, or external scripts/fonts without an explicit request.

## Git

- Development happens on the branch designated for the session (currently `claude/gifted-tesla-ercq0t`). Do not push to other branches without permission.
- Write clear, descriptive commit messages in the imperative mood.
