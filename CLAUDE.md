# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up that runs entirely in the browser. The whole game is a single self-contained file, `index.html`, containing inline CSS, markup, and one IIFE-wrapped `<script>` that draws to a `<canvas>` with the 2D context. There is no build step, no bundler, no package manager, and no external dependencies.

`README.md` is a placeholder and contains no project documentation.

## Running and testing

- Open `index.html` directly in a browser, or serve the directory with any static server (for example `python3 -m http.server`).
- There is no test suite, linter, or build command. Verify changes by playing the game in a browser, covering the affected path (menu, gameplay, boss, pause, game over, mobile touch layout).

## File layout (`index.html`)

Approximate line ranges, which will drift as the file changes:

- `<style>` (~10–360): layout, HUD pills, overlay card, touch controls, vignette/flash, keyframes, reduced-motion rules.
- Markup (~362–420): `#container`, canvas `#gameCanvas`, `#hud`, boss bar, `#topBar` icon buttons, `#touchControls`, `#overlay` menu with difficulty buttons and controls hint.
- `<script>` (~422 onward), inside a single `(function () { 'use strict'; ... })()`:
  - DOM references and storage keys
  - Adaptive performance (`quality` 0–2, driven by frame timing)
  - Config tables: `DIFFICULTIES`, `THEMES`, `ENEMY_TYPES`, `BOSSES`, `BOSS_PATTERNS`
  - Audio: `ensureAudio`, `beep`, `noiseBurst`, and the `sfx` object
  - HUD helpers: `renderHearts`, `renderBombs`, `showComboPop`, `showStageBanner`, `screenShake`
  - Layout: `BASE_W = 480`, `BASE_H = 720`, `resize()` scales the canvas to fit the viewport
  - Input: keyboard (`keys` map by `KeyboardEvent.code`), mouse, and touch joystick/buttons
  - Simulation: `resetGame`, `spawnEnemy`, `spawnBoss`, `fireWeapon`, `update(dt)`, `updateBoss`, `applyPowerUp`, `useBomb`, `loseLife`, `endGame`, `togglePause`
  - Rendering: `drawPlane`, `drawBossShip`, `drawClouds`, `draw()`
  - Main loop: `loop(now)` via `requestAnimationFrame`, plus the start-button handler

## Game design reference

- **Difficulty** (`DIFFICULTIES`): `easy`, `normal` (default), `hard`. Each sets spawn timing, enemy speed, enemy fire chance, starting lives, starting bombs, and a boss HP multiplier.
- **Stages**: `KILLS_PER_STAGE = 18` kills advance the stage. Each stage triggers a boss fight with a health bar. Bosses cycle through `BOSSES`, and each has its own attack pattern set in `BOSS_PATTERNS`. Stage backgrounds cycle through `THEMES`.
- **Enemies** (`ENEMY_TYPES`): `grunt`, `interceptor` (weaving), and `tank` (multi-hit, unlocked at stage 2).
- **Weapons**: `vulcan` and `laser`, each with power levels 1–3 that change the spread pattern. Weapon power-ups (type `weapon`) raise the weapon level; `applyPowerUp` toggles between `vulcan` and `laser` at that point.
- **Power-ups**: `life`, `bomb`, and `weapon`.
- **Bombs**: limited per difficulty. Activated with X, Shift, the BOMB button, or the `useBomb` path. They clear the screen and damage bosses.
- **Score**: a kill-chain multiplier (`addScore`, `breakChain`) boosts points.

## Controls

- Keyboard: Arrows or WASD to move; Space to fire; X or Shift for bomb; P or Escape to pause.
- Mouse: move to steer; hold the mouse button to fire.
- Touch: drag in the left zone for the virtual joystick; tap FIRE and BOMB.

## Persistence

All state is kept in `localStorage` under the `skyfighter.*` prefix:

| Key | Meaning |
| --- | --- |
| `skyfighter.bestScore` | Best score (number) |
| `skyfighter.muted` | `'1'` when audio is muted |
| `skyfighter.difficulty` | `easy`, `normal`, or `hard` |
| `skyfighter.reducedMotion` | `'1'` or `'0'`; defaults to the `prefers-reduced-motion` media query when unset |

Reads and writes are not wrapped in try/catch, so keep that in mind if you add more storage usage. Browsers in private mode or with storage blocked can throw.

## Conventions

- Keep everything in `index.html` unless there is a clear reason to split files. A split would require updating this section.
- Do not add external CDN scripts, fonts, or npm dependencies without a discussion. The game is intentionally standalone.
- Keep the existing style: `const` and `let`, arrow functions, 2-space indentation, and `'use strict'` inside the IIFE.
- Constants for tuning live in the config tables near the top of the script. Prefer editing those over hard-coding magic numbers in update or draw logic.
- Accessibility matters here: respect `reducedMotion` for screen shake, flash, and pulse effects; keep the `#srLive` announcements in sync with significant state changes (stage clear, boss start, game over); keep the touch layout working at phone width.
- Maintain the performance fallback. New per-frame effects should degrade when `quality` is above 0.
- Canvas coordinates use the fixed `BASE_W × BASE_H` logical size; CSS scales the element. Do not use `window.innerWidth` for game-space math.

## Git

- Development branch for this work: `claude/gifted-tesla-o88c4u`.
- Write clear, descriptive commit messages, one logical change per commit.
