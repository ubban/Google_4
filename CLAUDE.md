# CLAUDE.md

Guidance for Claude Code when working in this repository.

## What this repo is

A single-page browser game, **Sky Fighter**: a Raiden-style vertical shoot-'em-up. The whole game lives in one file, `index.html` (about 1,670 lines), with inline CSS in `<style>` and inline JavaScript in one IIFE inside `<script>`. `README.md` is only a title stub.

There is no build step, no package manager, no bundler, no test suite, and no linter configured. Open `index.html` in a browser to run it.

## Layout of `index.html`

- **Lines ~10–362 (`<style>`)**: all CSS. Layout is a full-viewport flex container (`#container`) that centers a `<canvas id="gameCanvas">`. HUD elements (`#hud`, `.stat`), the overlay, touch controls, and the boss bar are absolutely positioned on top of it.
- **Lines ~363–421 (markup)**: canvas, HUD, start/pause/game-over overlay, touch controls (`#moveZone`, `#fireBtn`, `#bombBtn`), and the difficulty buttons (`.diffBtn`).
- **Lines ~422–1668 (`<script>`)**: one `(function () { 'use strict'; ... })();` IIFE. Nothing is exposed on `window`. Keep new code inside it.

### Script sections, in order

1. DOM lookups (`const xxxEl = document.getElementById(...)`).
2. Storage keys (`skyfighter.*` in `localStorage`) and the `announce()` screen-reader helper.
3. Adaptive quality: `trackPerf()` lowers the `quality` level (0–2) when average frame time is slow.
4. Tuning tables: `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES` (per-stage sky palettes), `ENEMY_TYPES`, `BOSSES`.
5. Audio: `ensureAudio()`, `beep()`, `noiseBurst()`, and the `sfx` object. Audio is Web Audio, synthesized in code, with no audio files.
6. HUD helpers: `renderHearts`, `renderBombs`, `flashHit`, `showStageBanner`, `screenShake`, and so on.
7. Input: keyboard (`keys[e.code]`), virtual joystick, touch buttons, and mouse steering on the canvas.
8. Game state: `resetGame()` builds the `player` and the arrays of enemies, bullets, and power-ups.
9. Drawing: `drawPlane`, `drawBossShip`, `drawClouds`, and the main `draw()`.
10. Simulation: `update(dt)` (player movement, firing, collisions, spawning, scoring), `updateBoss(dt)`, and the boss attack patterns in `BOSS_PATTERNS` (`fireRadial`, `fireAimedSpread`, `fireCross`, `fireHomingSeeded`, and so on).
11. Flow: `loseLife()`, `killBoss()`, `endGame()`, `togglePause()`.
12. `loop(now)` drives the frame, uses `dt` clamped to 0.05 s, and runs `update` and `draw` only when running and not paused.

## Game rules worth knowing

- Controls: mouse moves and clicks to fire, Arrows or WASD move, Space fires, X or Shift drops a bomb, P or Esc pauses. Touch uses the on-screen joystick, FIRE, and BOMB.
- Difficulty (`easy` / `normal` / `hard`) sets spawn rates, enemy speed, shot chance, starting lives and bombs, and a boss HP multiplier.
- Killing `KILLS_PER_STAGE` (18) enemies advances the stage. Each stage uses the next `THEMES` entry and the next `BOSSES` entry.
- Weapons: `vulcan` and `laser`. Weapon power-ups toggle between them and raise `level`.
- Persisted in `localStorage`: best score, mute, difficulty, and reduced motion. Reduced motion defaults to the `prefers-reduced-motion` media query.

## Conventions to follow

- **Keep it one file.** Do not split out modules, add a build step, or add dependencies unless the user asks. Everything stays in `index.html`.
- **Match the existing style**: 2-space indent, single quotes in JS, `const` and `let` (no `var`), arrow callbacks, and short section banners in the form `// ---------- Name ----------`.
- **Tune with the tables.** Difficulty, enemy stats, boss stats, and stage colors belong in the constant objects at the top of the script, not as magic numbers inside `update()` or `draw()`.
- **Frame-rate independence.** Movement and timers multiply by `dt` (seconds). Don't add per-frame increments without it.
- **Storage access**: read and write `localStorage` only through the `STORAGE_*` constants, and keep the `skyfighter.` prefix.
- **Accessibility**: respect `reducedMotion` for screen shake, flashes, and similar effects, and send status messages through `announce()`.
- **Mobile**: the layout uses `touch-action: none`, safe-area insets, and a `resize()` that scales a fixed `BASE_W` × `BASE_H` (480 × 720) logical canvas. Check that changes still work at phone width.

## Verifying changes

There are no automated tests or lint scripts. To verify a change:

1. Open `index.html` in a browser (or serve the folder with any static server) and check the browser console for errors.
2. Play through the start, pause, game-over, and restart flow, and check each difficulty.
3. Check touch controls with device emulation, and keyboard and mouse controls on desktop.
4. Check that a stage clears and the boss fight, stage banner, and boss HP bar all appear.

If you add a test or lint setup, document the command here.

## Git

- Development branch for this work: `claude/gifted-tesla-xjr7qi`.
- Write clear commit messages that describe the gameplay or code change.
