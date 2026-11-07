# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo is **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs in the browser. The entire game is a single self-contained file:

- `index.html` — markup, inline `<style>`, and one inline `<script>` (an IIFE). No build step, no bundler, no external dependencies or CDN links.
- `README.md` — minimal placeholder.

There is no package.json, test suite, linter, or CI configuration. Verify changes by opening `index.html` in a browser (or serving the directory with any static server) and playing.

## Architecture (inside `index.html`)

The script is organized roughly in this order:

1. **DOM refs and constants** — element lookups, `STORAGE_*` localStorage keys (`skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion`).
2. **Tuning tables** — `DIFFICULTIES` (easy/normal/hard: spawn rates, enemy speed and shooting odds, lives, bombs, boss HP multiplier), `THEMES` (per-stage sky palettes, cycled by stage), `ENEMY_TYPES` (grunt, interceptor, tank), `BOSSES` (six bosses, cycled per stage), `KILLS_PER_STAGE`.
3. **Audio** — `ensureAudio`, `beep`, `noiseBurst`, and an `sfx` object. Audio is lazily initialized on first user gesture.
4. **HUD / feedback** — `renderHearts`, `renderBombs`, combo pop, stage banner, screen shake, hit/bomb flashes.
5. **Input** — keyboard (`keydown`/`keyup`), mouse on the canvas, and touch controls (virtual joystick + fire/bomb buttons).
6. **Game state and logic** — `resetGame`, `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`, `applyPowerUp`, `fireWeapon`, `useBomb`, `update(dt)`, `loseLife`, `endGame`, `togglePause`.
7. **Boss patterns** — `BOSS_PATTERNS` (radial, aimed spread, columns, spiral, cross, homing, double ring) driven by `updateBoss`.
8. **Rendering** — `draw()` and helpers (`drawPlane`, `drawBossShip`, `drawClouds`).
9. **Main loop** — `loop(now)` via `requestAnimationFrame`, started at the bottom of the script after the start button wiring.

Key gameplay facts to keep in mind when editing:

- **Weapons:** two types, `vulcan` (spread shots) and `laser` (pierce). Weapon power-ups raise `player.level` up to 3; past level 3 they toggle the weapon type and reset the level to 1.
- **Lives and bombs** are capped by `maxLives` / `maxBombs`, which come from the selected difficulty.
- **Stages:** after `KILLS_PER_STAGE` kills a boss warning fires and the boss spawns (`stageKills`, `bossWarned`); the stage's theme and boss change as stages advance.
- **Best score** is persisted to localStorage only when a game ends with a new high score.

## Conventions

- Keep everything in `index.html` unless there is a clear reason to split files. If you do split, update this file.
- Plain JavaScript, ES2015+ syntax already in use (arrow functions, `const`/`let`, template literals). Do not introduce a build tool or framework without being asked.
- Follow the existing style: 2-space indentation, single quotes in JS, `'use strict'` inside the IIFE, and small named helper functions.
- Keep game-tuning numbers in the existing constant tables (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `THEMES`) rather than hard-coding them in logic.
- Keep the UI usable on phones: the page uses `touch-action: none`, a viewport meta with `user-scalable=no`, and responsive `clamp()` sizing. Preserve these when changing layout.
- Accessibility features already present should be kept working: the `srLive` aria-live region (`announce()`), the reduced-motion toggle (which affects screen shake and flashes), and the mute toggle.
- Do not add analytics, network calls, or third-party scripts.

## Working in this repo

- Develop on the designated feature branch (currently `claude/gifted-tesla-cwilaz`) and push with `git push -u origin <branch>`.
- Commit messages should be short and descriptive of the gameplay or code change.
- Before finishing a change, open the game in a browser and check at least: start screen, a full play-through of a stage, pause/resume, game over with best-score update, and mobile-width layout.
- Do not create pull requests unless explicitly asked.
