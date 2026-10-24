# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo is **Sky Fighter**, a single-file, browser-based Raiden-style vertical shoot-'em-up. There is no build step, package manager, or test suite.

- `index.html` — the entire game: markup, CSS (inline `<style>`), and JavaScript (inline `<script>`, wrapped in an IIFE with `'use strict'`).
- `README.md` — placeholder; contains no project documentation.

Open `index.html` directly in a browser to play. No server is required.

## Repository layout (inside `index.html`)

- **Lines ~1–360** — `<head>` meta tags and inline CSS.
- **Lines ~360–420** — HTML for the canvas, HUD (score, best, multiplier, weapon, hearts, bombs, boss bar), overlays, touch controls, and accessibility live region (`#srLive`).
- **Lines ~422 onward** — the game script. Roughly in order:
  - DOM lookups and `STORAGE_*` keys (`localStorage` under the `skyfighter.*` prefix).
  - Config constants: `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES`, `ENEMY_TYPES`, `BOSSES`.
  - Adaptive performance (`trackPerf` / `quality`), which lowers render quality when frame time is high.
  - Web Audio SFX (`ensureAudio`, `beep`, `noiseBurst`, `sfx`).
  - Input: keyboard, virtual joystick and buttons for touch, and mouse on the canvas.
  - Game state and logic: `resetGame`, `spawnEnemy`, `spawnBoss`, `update`, `fireWeapon`, `useBomb`, `applyPowerUp`, `BOSS_PATTERNS`, `updateBoss`, `loseLife`, `endGame`, `togglePause`.
  - Rendering: `draw`, `drawPlane`, `drawBossShip`, `drawClouds`.
  - Main loop: `loop(now)` runs `update` then `draw` each frame, with `dt` clamped to 0.05s.

Use `grep -n "function name" index.html` to locate code. Line numbers drift as the file changes, so search rather than trusting them.

## Conventions

- **Keep it one file.** The game is deliberately self-contained. Do not split out modules, add a bundler, or add external dependencies unless the user asks.
- **Match the existing style.** Two-space indentation, single quotes in JS, `const` by default, arrow functions for callbacks, and section banner comments like `// ---------- Name ----------`.
- **Tune gameplay through the config tables.** Difficulty, enemy stats, boss stats, and theme colors live in the constant objects near the top of the script. Prefer editing those over hard-coding numbers in logic.
- **Persist settings only through the `skyfighter.*` keys.** Wrap `localStorage` access so the game still runs if storage is unavailable.
- **Support both input methods.** Any new control needs keyboard, touch, and (where relevant) mouse handling, and should respect the `reducedMotion` setting for new visual effects like shake or flash.
- **Announce important state changes** through `announce()` so screen readers pick them up.
- **Use `dt`-based movement.** Frame-rate-independent updates use the `dt` passed to `update(dt)`; do not assume a fixed frame rate.

## Verifying changes

There is no automated test or lint setup. To verify a change:

1. Open `index.html` in a browser (or serve the directory with any static server).
2. Play through at least one stage, including a boss fight, and check the affected behavior.
3. Check the console for errors. The game is wrapped in an IIFE, so errors surface only in the console.
4. For layout or input changes, test at phone width and with touch emulation, since the game has touch controls and a `resize()` scaler based on a 480×720 base.

If you add a test or lint setup, document the commands here.

## Git workflow

- Develop on the branch the session specifies. Do not push to `main` without explicit permission.
- Write clear, descriptive commit messages that explain *why* a change was made.
- Do not open pull requests unless the user asks for one.
