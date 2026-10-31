# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo is a single-page HTML5 canvas game, **Sky Fighter** — a Raiden-style vertical shoot-em-up.

- `index.html` (~1,670 lines) is the entire game: inline `<style>`, the HTML/HUD/overlay markup, and one inline `<script>` wrapped in an IIFE (`'use strict'`).
- `README.md` is a placeholder with no setup instructions.
- There is no build step, package manager, bundler, test suite, linter, or CI config. Nothing is installed; there are no dependencies.

## Running and verifying

- Open `index.html` directly in a browser, or serve the directory with any static server (e.g. `python3 -m http.server`) and visit it.
- There are no automated tests. Verify changes by playing the game in a browser, ideally at both desktop and mobile-sized viewports, and checking the console for errors.

## Code layout inside `index.html`

The script is organized roughly in this order. Search for these names to navigate:

1. **DOM references** — `const xxxEl = document.getElementById(...)` at the top of the IIFE.
2. **Storage keys** — `STORAGE_BEST`, `STORAGE_MUTE`, `STORAGE_DIFF`, `STORAGE_REDUCED` (all under the `skyfighter.*` prefix in `localStorage`).
3. **Tuning tables** — `DIFFICULTIES` (spawn rate, enemy speed/fire chance, lives, bombs, boss HP multiplier), `KILLS_PER_STAGE`, `THEMES` (per-stage sky palettes), `ENEMY_TYPES` (grunt / interceptor / tank), `BOSSES` (six boss designs), `BOSS_PATTERNS` (bullet patterns).
4. **Adaptive quality** — `trackPerf()` steps a `quality` level (0–2) up or down based on a rolling frame-time average.
5. **Audio** — `ensureAudio()`, `beep()`, `noiseBurst()`, and the `sfx` table. Audio is created lazily on the first user gesture.
6. **Input** — keyboard (arrows/WASD, Space fire, X or Shift bomb, P/Esc pause), mouse (move to steer, click to fire), and touch (joystick on the left, FIRE and BOMB buttons).
7. **Game state and logic** — `resetGame()`, `update(dt)`, `spawnEnemy()`, `spawnBoss()`, `applyPowerUp()`, `useBomb()`, `fireWeapon()`, `updateBoss()`, `killBoss()`, `loseLife()`, `endGame()`, `togglePause()`.
8. **Rendering** — `draw()`, `drawPlane()`, `drawBossShip()`, `drawClouds()`, plus the parallax layers.
9. **Main loop** — `loop(now)` uses `requestAnimationFrame` with a clamped `dt`.

Coordinates use a fixed logical canvas of **480 × 720** (`BASE_W`, `BASE_H`); `resize()` scales it to the viewport.

## Game design facts to preserve

- **Weapons:** Vulcan and Laser, each with 3 power levels. Collecting a weapon power-up raises the level; past level 3 it toggles the weapon type and resets to level 1.
- **Power-ups:** `life` (up to `maxLives`), `bomb` (up to `maxBombs`), and `weapon`.
- **Bombs:** limited smart bombs that clear enemy bullets and enemies and damage bosses.
- **Stages:** waves of enemies; a stage ends after `KILLS_PER_STAGE` kills and leads into a boss. Enemy types unlock by `minStage`.
- **Score:** a kill-chain multiplier (`addScore`, `breakChain`).
- **Difficulty:** `easy`, `normal`, `hard`, chosen on the start overlay and persisted.
- **Accessibility:** a reduced-motion toggle (screen shake, flashes, and pulses) that defaults to the `prefers-reduced-motion` media query. Screen-reader announcements go through `announce()` into the `#srLive` live region.

## Conventions

- Keep everything in `index.html` unless there is a clear reason to split it. Adding a build step or external dependencies is a significant change; ask first.
- Keep the code in the existing style: `const`/`let`, the IIFE scope, no globals, and two-space indentation.
- Balance changes go in the tuning tables (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `BOSS_PATTERNS`) rather than being hard-coded in logic.
- Any new persistent setting should use a `skyfighter.*` localStorage key, and reads and writes should tolerate unavailable storage.
- Keep the mobile layout working: touch controls are shown on touch devices and must not block the HUD or the playfield.
- Preserve the `prefers-reduced-motion` behavior when adding screen shake, flashes, or pulses.

## Git workflow

- Develop on the branch designated for the task (currently `claude/gifted-tesla-pwrmc2`). Do not push to `main` without explicit permission.
- Write clear, descriptive commit messages in the imperative mood.
- Do not open a pull request unless explicitly asked.
