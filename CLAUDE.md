# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo contains **Sky Fighter**, a single-page, Raiden-style vertical shoot-'em-up built on an HTML5 canvas. The whole game lives in one file, `index.html`: markup (~lines 1–360), CSS (inline `<style>`), and one inline `<script>` IIFE (~lines 422–1667) with the game loop, rendering, audio, input, and persistence. `README.md` is a placeholder.

There is no build step, no package manager, no test suite, and no linter configured. Nothing needs to be installed.

## Running and verifying

- Open `index.html` directly in a browser, or serve the directory with any static server (e.g. `python3 -m http.server`) and load it.
- Verify changes by playing in a browser: check the title overlay, a full run, the boss fight, pause, game over, and the mobile touch controls when touching input code.
- Use the browser console to catch runtime errors. The game runs inside an IIFE with `'use strict'`, so nothing is exposed on `window` for debugging.

## Code layout inside `index.html`

The script is organized by section comments. Keep new code in the matching section:

- **DOM references and constants** (top of the IIFE): `STORAGE_*` keys, `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES`, `ENEMY_TYPES`, `BOSSES`.
- **Adaptive performance** (`trackPerf`): samples frame time and steps a `quality` level (0 full, 1 medium, 2 low) that drives visual detail.
- **Audio** (`ensureAudio`, `beep`, `noiseBurst`, `sfx`): Web Audio synthesis only; no audio files. Respects `muted`.
- **HUD helpers**: `renderHearts`, `renderBombs`, `showComboPop`, `showStageBanner`, `screenShake`, `flashHit`/`flashBomb`, `announce` (screen-reader live region).
- **Input**: keyboard (`keys` map), touch joystick and buttons, mouse aim via `canvasPointFromEvent`.
- **Game state and flow**: `resetGame`, `update(dt)`, `applyPowerUp`, `useBomb`, `fireWeapon`, `loseLife`, `endGame`, `togglePause`.
- **Enemies and bosses**: `spawnEnemy`, `spawnBoss`, `updateBoss`, `killBoss`, and the `BOSS_PATTERNS` array of bullet-pattern functions (`fireRadial`, `fireAimedSpread`, `fireCross`, etc.).
- **Rendering**: `draw()`, `drawPlane`, `drawBossShip`, `drawClouds`. Drawing is procedural; there are no image or sprite assets.
- **Loop**: `loop(now)` clamps `dt` to 0.05 s, calls `update` then `draw` only while running and not paused, then re-queues itself.

## Conventions

- **Language and style**: plain ES2015+ JavaScript, `const`/`let`, arrow functions for callbacks, two-space indentation, single quotes, semicolons. Match the existing style of the surrounding code.
- **Single file**: keep everything in `index.html` unless the user asks for a split. Do not add a bundler, framework, or dependencies.
- **Canvas coordinates**: the logical play area is `BASE_W` × `BASE_H` (480 × 720). `resize()` scales it to the viewport. Use logical coordinates in game code.
- **Frame-rate independence**: movement and timers use `dt` (seconds). Don't hard-code per-frame increments.
- **Constants over magic numbers**: put tunable values (speeds, spawn rates, HP, score values, difficulty multipliers) in the existing constant tables rather than inline.
- **Persistence**: all saved state goes through `localStorage` under the `skyfighter.*` key prefix, and every read/write must tolerate the value being absent. Current keys: `bestScore`, `muted`, `difficulty`, `reducedMotion`.
- **Accessibility**: respect `reducedMotion` (disables or reduces screen shake and flashes). Announce important state changes through `announce()`. Keep keyboard controls working.
- **Adding enemies, bosses, or themes**: extend the relevant table (`ENEMY_TYPES`, `BOSSES`, `THEMES`) and the matching spawn/draw/pattern code. Check `minStage` gating and boss rotation (boss index is chosen from the stage number).

## Gameplay reference

- Difficulty (`easy` / `normal` / `hard`) sets spawn cadence, enemy fire chance, enemy speed, starting lives and bombs, and boss HP multiplier.
- Weapons: the weapon power-up raises the current weapon from level 1 to 3; the next pickup at level 3 switches between Vulcan and Laser and resets to level 1 (`applyPowerUp`).
- Other power-ups: extra life (capped at `maxLives`) and bomb (capped at `maxBombs`).
- Bombs clear all enemies and enemy bullets, and deal 24 damage to a boss. Starting bomb count comes from difficulty.
- Score uses a kill-chain multiplier (`addScore`, `breakChain`).
- After `KILLS_PER_STAGE` kills, the stage's boss is announced and spawned. Boss `design` is `BOSSES[(stage - 1) % BOSSES.length]`, and its attack pattern comes from `BOSS_PATTERNS` by the same index. Boss HP is `(70 + stage × 35) × difficulty.bossHpMul × design.hpMul`.

## Git workflow

- Develop on the branch the session specifies (currently `claude/gifted-tesla-tizduv`). Do not push to other branches without permission.
- Write clear, imperative commit messages that describe the gameplay or code change.
- Do not open a pull request unless the user asks for one.

## When editing this file

Keep CLAUDE.md accurate to the code. If you change the section layout of `index.html`, a storage key, a difficulty value, or a constant table, update the relevant lines here in the same change.
