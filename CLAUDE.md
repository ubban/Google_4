# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo is **Sky Fighter**, a single-file, browser-based vertical shoot-'em-up (Raiden-style). There is no build step, no package manager, and no test suite.

Files:
- `index.html` (~1,670 lines): the entire game. Inline `<style>` (lines ~8–360), HTML markup (lines ~361–421), and one IIFE-wrapped `<script>` (lines ~422–1667).
- `README.md`: a one-line title only. It currently contains a stray timestamp, not real documentation.

## Running and verifying

- Open `index.html` directly in a browser, or serve the folder with any static server (e.g. `python3 -m http.server`).
- There are no automated tests, linters, or build commands. Verify changes by playing the game in a browser, with particular attention to the area you changed.
- Browser test tooling, if needed, is available through the Playwright/Chromium setup in the remote environment.

## Code layout inside `index.html`

The script is one IIFE with `'use strict'`. Key sections, roughly in file order:

- **DOM references and storage keys**: `STORAGE_BEST`, `STORAGE_MUTE`, `STORAGE_DIFF`, `STORAGE_REDUCED` (all prefixed `skyfighter.`) are the `localStorage` keys.
- **Adaptive quality** (`trackPerf`, `quality` 0–2): lowers effect quality when frame time is high.
- **Config tables**:
  - `DIFFICULTIES` (`easy` / `normal` / `hard`): spawn rates, enemy speed and fire chance, lives, bombs, boss HP multiplier.
  - `KILLS_PER_STAGE` (18): kills needed to advance a stage.
  - `THEMES`: six sky palettes, cycled by stage.
  - `ENEMY_TYPES`: `grunt`, `interceptor` (weaving), `tank` (4 HP, unlocks at stage 2).
  - `BOSSES`: six bosses (`VANGUARD` → `TITAN`) with colors and `hpMul`.
- **Audio**: `ensureAudio`, `beep`, `noiseBurst`, and an `sfx` object. Audio is created lazily on the first user gesture, and mute state persists.
- **Rendering helpers**: `renderHearts`, `renderBombs`, `flashHit`, `screenShake`, `showComboPop`, `showStageBanner`, and the HUD DOM updates.
- **Layout and input**: `resize()` fits a fixed `BASE_W × BASE_H` (480 × 720) logical canvas into the viewport. Keyboard, mouse, and touch (joystick, FIRE, BOMB) handlers live here.
- **Game state**: `resetGame()` initializes `player`, `bullets`, `enemies`, `enemyBullets`, `powerUps`, and related arrays.
- **Entities and spawning**: `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`, `spawnExplosion`, `rectsOverlap` (AABB collision).
- **Weapons**:
  - `fireWeapon()`: `vulcan` (spread shots) or `laser` (piercing lanes), each with levels 1–3.
  - `applyPowerUp()`: `life`, `bomb`, and `weapon` pickups. Weapon pickups level up the current weapon to 3, then switch between vulcan and laser.
  - `useBomb()`: screen-clearing smart bomb.
- **Boss patterns**: `fireRadial`, `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`, `fireCross`, `fireHomingSeeded`, `fireDoubleRing`, assembled per boss in `BOSS_PATTERNS`. `updateBoss` and `killBoss` drive boss phases.
- **Main update and draw**: `update(dt)` advances simulation, `draw()` renders a frame, and `loop(now)` runs both via `requestAnimationFrame`. Each frame's `dt` is clamped to 50 ms.
- **Game flow**: `loseLife`, `endGame`, `togglePause`, and the start button handler.

## Controls (for reference when testing)

- Move: mouse, or Arrow keys / WASD.
- Fire: Space, or hold the mouse button, or FIRE on touch.
- Bomb: `X` or Shift, or BOMB on touch.
- Pause: `P` or Esc.

## Conventions

- Keep the game in a single self-contained `index.html` unless the user asks for a split. Do not add a bundler or framework.
- Preserve the existing style: `const`/`let`, arrow functions in callbacks, and the IIFE wrapper with `'use strict'`.
- Keep tuning values in the existing config tables (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `THEMES`) rather than hard-coding them in logic.
- Keep the UI usable on phones: the layout uses `clamp()` sizing, safe-area insets, and touch controls. Check mobile layout when changing HUD or overlay CSS.
- Honor the `reduced-motion` setting (`reducedMotion`, `body.reduced-motion`) for any new screen shake, flashes, or animation.
- Use `localStorage` only with the `skyfighter.` prefix, and wrap new storage access so the game still works if storage is unavailable.
- Accessibility: announce important state changes through the `announce()` live region.

## Git workflow

- Develop on the branch the session specifies (currently `claude/gifted-tesla-v85al8`). Do not push to `main` without explicit permission.
- Do not open a pull request unless the user asks for one.
