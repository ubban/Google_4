# CLAUDE.md

Guidance for Claude Code when working in this repository.

## What this is

Google_4 is a small, dependency-free browser game, **Sky Fighter** (a Raiden-style vertical shoot-em-up). The whole game is one file: `index.html`, containing inline CSS (`<style>`), the DOM markup (`<body>`), and one IIFE-wrapped `<script>` block. `README.md` is a placeholder.

There is no build step, package manager, test suite, linter, or CI configured. Do not add one unless asked.

## Running it

Open `index.html` directly in a browser, or serve the directory with any static server (for example `python3 -m http.server`). No install is needed.

To verify a change, load the page and play it. Check the canvas, the HUD (score, best, multiplier, weapon, hearts, bombs), the overlay and difficulty buttons, and the boss bar. There are no automated checks to run.

## Layout of `index.html`

- **Lines ~1–360, `<style>`**: all styling, including the responsive canvas sizing, HUD, overlay, touch controls, and the `.reduced-motion` class.
- **Lines ~361–421, `<body>`**: canvas `#gameCanvas`, HUD elements, overlay (`#overlay`), touch controls (`#touchControls`, `#moveZone`, `#fireBtn`, `#bombBtn`), and an `aria-live` region (`#srLive`).
- **Lines ~422–1668, `<script>`**: everything else, all inside one `(function () { 'use strict'; ... })()`. Nothing is exported and there are no modules.

Within the script, the sections are marked with `// ---------- Name ----------` comments. Follow that convention when adding sections. Main groups, in order:

1. **DOM refs and constants**: `STORAGE_*` keys, `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES` (background palettes per stage), `ENEMY_TYPES`, `BOSSES`.
2. **Persistence and settings**: best score, mute, difficulty, and reduced motion, all stored in `localStorage` under `skyfighter.*`.
3. **Audio**: a lazily created `AudioContext` (`ensureAudio`), with `beep` and `noiseBurst` helpers. Audio starts only after a user gesture.
4. **Input**: keyboard (`keys` map, `keydown`/`keyup` on `window`), mouse/pointer steering and firing, and a touch joystick plus fire and bomb buttons.
5. **Game state and lifecycle**: `resetGame`, `update(dt)`, `endGame`, `togglePause`, `loseLife`, and `killBoss`.
6. **Spawning and combat**: `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`, `applyPowerUp`, `fireWeapon`, `useBomb`, `addScore`/`breakChain` (the combo multiplier), and `BOSS_PATTERNS` (bullet-pattern functions such as `fireRadial`, `fireCross`, `fireHomingSeeded`).
7. **Rendering**: `draw()`, `drawPlane`, `drawBossShip`, `drawClouds`, and the screen-shake and flash effects.
8. **Main loop**: `loop(now)` is driven by `requestAnimationFrame`. It clamps `dt` to 0.05 s and calls `trackPerf` for adaptive quality.

## Conventions

- **Plain JavaScript, ES2015+ syntax**: `const`/`let`, arrow functions, and template literals. Keep to what the file already uses. No libraries and no transpilation.
- **Keep it in one file** unless the user asks for a split. Keep the existing IIFE and `'use strict'`. Do not leak globals.
- **Match the surrounding style**: two-space indentation, single quotes, semicolons, and short section banners. Keep comment density similar to the nearby code.
- **Constants live in the top-level config objects** (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `THEMES`). Prefer adding tuning values there over hard-coding them in logic.
- **Use `dt` (seconds) for movement and timers**, so behavior stays frame-rate independent.
- **Respect accessibility hooks**: announce important state changes through `announce()` (the `aria-live` region). Honor `reducedMotion` for shake, flash, and similar effects.
- **Persist settings through the `STORAGE_*` keys and the existing setter functions** (`setDifficulty`, `updateMuteBtn`, `updateMotionBtn`). Do not write to `localStorage` directly from elsewhere.
- **Performance**: the adaptive `quality` level (0 full, 1 medium, 2 low) reduces effects when frame times are slow. Keep new per-frame work cheap, and avoid allocating in hot paths where practical. Entities are plain objects in arrays (`bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`) that are filtered each frame.
- **Touch and desktop both matter.** Anything that adds an input should work with the keyboard and mouse as well as the touch controls. The `controlsHint` text lists the current bindings, so update it when bindings change.

## Current controls

- Move: mouse, or Arrow keys / WASD
- Fire: Space (hold), or click / hold the mouse, or the FIRE button on touch
- Bomb: X or Shift, or the BOMB button
- Pause: P or Esc
- Mute and reduced motion: on-screen buttons

## Gameplay reference

- Difficulty (`easy` / `normal` / `hard`) sets lives, bombs, spawn rate, enemy shot chance, and boss HP multiplier.
- Each stage lasts `KILLS_PER_STAGE` (18) kills, then a boss fight. Stages cycle through `THEMES`, and each boss comes from `BOSSES`.
- Enemy types: `grunt`, `interceptor` (weaving), `tank` (4 HP, appears from stage 2).
- Power-ups (`powerUps` array, `type`): `life`, `bomb`, `weapon`.
- Score is multiplied by a combo chain; the chain resets via `breakChain()`.
- The best score persists in `localStorage`.

## Working agreements

- Make the smallest change that does the job, and keep edits inside `index.html` unless the task needs another file.
- After editing, play-test the affected path in a browser: the change itself, plus a quick run through start, a boss, pause, game over, and restart.
- Commit with a short, imperative summary line describing the change (the existing history uses this style, e.g. "Add Sky Fighter: ...").
