# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo is **Sky Fighter**, a Raiden-style vertical shoot-'em-up, delivered as a single self-contained page: `index.html` (HTML + inline CSS + inline JavaScript, ~1,670 lines, rendered on a `<canvas>`). `README.md` is only a stub.

There is no build system, no package manager, no bundler, no test suite, and no linter. Do not add one unless the user asks.

## Running the game

- Open `index.html` directly in a browser, or serve the directory with any static server.
- For automated checks, use the pre-installed Playwright/Chromium (see the environment notes), not a fresh `playwright install`.

## Code layout (`index.html`)

- **`<style>` block (approx. lines 10–362):** all styling. Uses a dark `color-scheme`, a pill-style HUD (`#hud`), a boss bar (`#bossBarWrap`), touch controls (`#touchControls`), and the overlay card (`#overlay`).
- **`<body>` markup (approx. lines 363–421):** canvas `#gameCanvas`, HUD elements, top bar buttons (`#muteBtn`, `#motionBtn`, `#pauseBtn`), touch joystick/FIRE/BOMB, and the start/title overlay with difficulty buttons.
- **`<script>` block (starts at approx. line 422):** all game logic, wrapped in an IIFE-style scope. Key sections, in file order:
  - **DOM refs and constants:** element lookups, `STORAGE_*` localStorage keys, `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES` (per-stage sky palettes), `ENEMY_TYPES`, `BOSSES`.
  - **Settings:** best score, mute, difficulty, and reduced-motion preferences are persisted to `localStorage`.
  - **Audio:** lazily created `AudioContext` (`ensureAudio`), with `beep`, `noiseBurst`, and a named `sfx` table. Audio starts only after a user gesture.
  - **Performance:** `trackPerf` adjusts a `quality` level (0 full / 1 medium / 2 low) based on frame times.
  - **Input:** keyboard (`keys`), touch joystick, fire/bomb buttons, and mouse steering on the canvas.
  - **Game state:** `resetGame()` initializes entities (`player`, `bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`, `boss`) and counters (score, lives, bombs, stage, kill chain/multiplier).
  - **Rendering:** `drawPlane`, `drawBossShip`, `drawClouds`, and `draw()`.
  - **Spawning and combat:** `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`, `fireWeapon`, `useBomb`, the boss pattern functions (`fireRadial`, `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`, `fireCross`, `fireHomingSeeded`, `fireDoubleRing`), and `updateBoss`/`killBoss`.
  - **Flow:** `update(dt)`, `applyPowerUp`, `loseLife`, `endGame`, `togglePause`, and `loop(now)` (the `requestAnimationFrame` main loop).

## Game design reference

Keep these values consistent when changing gameplay:

- **Difficulty** (`DIFFICULTIES`): `easy`, `normal` (default), `hard`. Each sets spawn timing, enemy speed, enemy fire chance, starting lives and bombs, and a boss HP multiplier.
- **Stages:** advance after `KILLS_PER_STAGE` (18) kills. Each stage ends in a boss from `BOSSES`, indexed by `(stage - 1) % BOSSES.length`. Background theme comes from `THEMES`, indexed the same way.
- **Enemies:** `grunt` (stage 1+), `interceptor` (stage 1+, weaving), `tank` (stage 2+, 4 HP).
- **Weapons:** Vulcan and Laser, each with 3 power levels.
- **Scoring:** a kill chain raises the score multiplier; breaking the chain resets it.
- **Lives and bombs:** smart bombs clear the screen and damage bosses.

## Conventions

- **Keep it a single file** unless the user asks for a split. Preserve the self-contained, no-dependency nature of the page.
- **No external network dependencies.** Do not add CDN scripts, fonts, or assets without an explicit request.
- **Preserve the storage keys** (`skyfighter.*`). Changing them silently resets players' saved best score, mute, difficulty, and motion settings.
- **Respect the accessibility features:** reduced-motion toggle (screen shake, flashes, pulses), the `#srLive` aria-live announcer, and the mobile touch controls. Changes to HUD or overlay should keep the page usable at phone width (the viewport meta disables zoom).
- **Keep the canvas logical size** at `BASE_W × BASE_H` (480 × 720). `resize()` scales it to fit the window.
- **Match existing style:** 2-space indentation, single quotes in JS, short helper functions, and no frameworks.

## Git workflow

- Develop on the branch the session names (currently `claude/gifted-tesla-46luiv`). Do not push to other branches without permission.
- Commit with clear messages describing the gameplay or UI change.
- Do not open a pull request unless the user explicitly asks for one.
