# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Google_4 is a single-page HTML5 canvas game called **Sky Fighter**, a Raiden-style vertical shoot-'em-up. The whole game lives in `index.html`: markup, CSS (`<style>`, lines ~10–359), and one IIFE-wrapped `<script>` (lines ~422–1667). `README.md` is a stub.

There is no build step, package manager, bundler, test suite, or linter. Do not add one unless asked.

## Running and verifying

- Open `index.html` directly in a browser, or serve the folder with any static server.
- Verification is manual: play the game in a browser and check the behavior you changed. There are no automated tests to run.
- Check both desktop (mouse and keyboard) and touch layouts when touching UI, since the touch controls only show on touch devices.

## Code layout (`index.html`)

- **Config constants** (top of the script): `DIFFICULTIES` (easy/normal/hard spawn rates, enemy speed, shot chance, lives, starting bombs, boss HP multiplier), `KILLS_PER_STAGE` (18), `THEMES` (per-stage sky palettes), `ENEMY_TYPES` (grunt, interceptor, tank), `BOSSES` (six designs, cycled by stage), and `BOSS_PATTERNS` (bullet patterns, indexed by boss design).
- **Persistence**: `localStorage` keys prefixed `skyfighter.` (`bestScore`, `muted`, `difficulty`, `reducedMotion`). Every read and write should tolerate missing or blocked storage.
- **Game state and loop**: `resetGame()` initializes state, `update(dt)` advances simulation, `draw()` renders, and `loop(now)` drives both through `requestAnimationFrame`.
- **Weapons and power-ups**: `fireWeapon()` and `applyPowerUp()`. The Vulcan/Laser system has 3 power levels.
- **Bombs and chain score**: `useBomb()` clears the screen and damages bosses. `addScore()` and `breakChain()` manage the kill-chain multiplier.
- **Bosses**: `spawnBoss()`, `updateBoss()`, `killBoss()`, with the attack pattern functions (`fireRadial`, `fireAimedSpread`, `fireHomingSeeded`, and others) referenced from `BOSS_PATTERNS`.
- **Audio**: a Web Audio synth (`beep`, `noiseBurst`). Audio starts on first user gesture via `ensureAudio()`.
- **Adaptive quality**: `trackPerf()` steps a `quality` level (0–2) based on average frame time over 45 frames.
- **Input**: keyboard (`keys` map, listeners on `window`), mouse/pointer on the canvas, and a virtual joystick and buttons for touch.
- **Accessibility**: `announce()` writes to a live region, and a reduced-motion setting is honored, including the `prefers-reduced-motion` media query.

## Conventions

- Keep everything in the single file unless the user asks for a split. Match the existing style: 2-space indentation, `const`/`let`, arrow functions for callbacks, and short section comments like `// ---------- Name ----------`.
- Keep the game loop allocation-light. Reuse arrays and objects where the surrounding code already does.
- Use the existing `DIFFICULTIES` and `BOSSES` tables for tuning values rather than hardcoding numbers in logic.
- Tune gameplay through the config constants first. Change logic only when a constant can't express the change.
- Preserve the `'use strict'` IIFE wrapper and avoid new globals.

## Git

- Development branch: `claude/gifted-tesla-yiiox1`. Push with `git push -u origin <branch>`.
- Commit messages describe the change in plain terms. Recent history uses short imperative subjects, e.g. "Add Sky Fighter: ...".
