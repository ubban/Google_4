# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Sky Fighter is a Raiden-style vertical shoot-'em-up that runs in the browser. The whole game lives in one file, `index.html` (about 1,670 lines): inline CSS in `<style>`, markup in `<body>`, and all JavaScript in a single IIFE inside `<script>`. `README.md` is a placeholder.

There is no build step, package manager, bundler, test suite, or linter. Do not add one unless the user asks.

## Running and checking changes

- Open `index.html` directly in a browser, or serve the directory with any static server.
- Verify behavior in a real browser, because the game is entirely visual and audio-driven. Check desktop (mouse and keyboard) and, when touching input or layout, a narrow touch viewport.
- Syntax-check the inline script after edits, for example by extracting the `<script>` block and running `node --check` on it.

## Code layout inside `index.html`

Sections appear in this order inside the IIFE, marked by `// ---------- Name ----------` comments where they exist:

1. **DOM references**: `const xxxEl = document.getElementById(...)`. Add new HUD or overlay elements here.
2. **Storage keys**: `STORAGE_*` constants with the `skyfighter.` prefix. Best score, mute, difficulty, and reduced motion persist in `localStorage`.
3. **Adaptive performance**: `trackPerf` lowers a `quality` level (0 full, 1 medium, 2 low) when frame time is high.
4. **Tuning data**: `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES` (per-stage sky palettes), `ENEMY_TYPES`, and `BOSSES`. Balance changes usually belong here, not in the logic.
5. **Audio**: `ensureAudio`, `beep`, and `noiseBurst`. Audio is created lazily on the first user gesture because browsers require one.
6. **Game state and actions**: `resetGame`, `spawnEnemy`, `spawnBoss`, `fireWeapon`, `useBomb`, `applyPowerUp`, `addScore`, `loseLife`, `endGame`, and `togglePause`.
7. **Boss attacks**: `BOSS_PATTERNS` and the `fire*` helpers (radial, spread, columns, spiral, cross, homing, double ring). `updateBoss` and `killBoss` drive boss phases.
8. **Input**: keyboard handlers on `window`, mouse on the canvas, and touch controls (a left-side joystick zone plus FIRE and BOMB buttons).
9. **Update and draw**: `update(dt)` advances the simulation. `draw()` renders everything onto the canvas. `drawPlane`, `drawBossShip`, and `drawClouds` are the sprite helpers.
10. **Main loop**: `loop(now)` clamps `dt` to 0.05 s, then calls `update` and `draw` only while the game is running and not paused.

## Conventions

- Keep everything in `index.html` unless the user asks for a split. Keep the IIFE and `'use strict'`.
- Use `const` by default and `let` for mutable state. Use arrow functions for callbacks, and match the surrounding style (two-space indent, single quotes in JavaScript).
- Add new tuning values to the existing data objects rather than hard-coding magic numbers in logic.
- Reuse the existing helpers (`rectsOverlap`, `spawnExplosion`, `screenShake`, `flashHit`, `announce`) instead of duplicating them.
- Respect reduced motion. Effects that shake the screen or flash should check the `reducedMotion` flag, the same as existing code.
- Keep HUD text and accessibility in step: update `announce()` for significant state changes so screen readers hear them.
- Keep the game playable on touch. Any new control needs a touch equivalent.
- Wrap `localStorage` access in a way that still works if storage is unavailable, such as a private window or blocked site data. The current code calls it directly, so new storage use should at least not throw.

## Git

- Development happens on the branch named in the session instructions. Do not push to `main` unless the user asks.
- Keep commits focused and write descriptive messages that say what changed and why.
