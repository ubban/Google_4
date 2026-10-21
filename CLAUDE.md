# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Sky Fighter is a Raiden-style vertical shoot-'em-up that runs entirely in the browser. The whole game is one self-contained file, `index.html`: markup, CSS, and JavaScript inline. `README.md` is a placeholder.

There is no build step, no package manager, no bundler, no framework, and no test suite. Do not add any of these unless the user asks.

## Repository layout

- `index.html` (~1,670 lines): the entire game.
  - Lines ~1–421: `<head>` metadata, inline `<style>`, and the DOM: `#container`, the `<canvas id="gameCanvas">`, the HUD, the boss bar, the touch controls, and the `#overlay` start/game-over card.
  - Line ~422 onward: one inline `<script>` containing a single IIFE (`(function () { 'use strict'; ... })();`). All game state is private to that closure.
- `README.md`: placeholder.

To run the game, open `index.html` in a browser. Nothing needs to be installed.

## Architecture of the game script

The IIFE is organized into sections, roughly in this order:

1. **DOM refs and constants.** Element lookups, `STORAGE_*` localStorage keys, and the gameplay tables: `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES`, `ENEMY_TYPES`, `BOSSES`.
2. **Persistence and settings.** Best score, mute, difficulty, and reduced-motion flags are read from and written to `localStorage`.
3. **Adaptive performance.** `trackPerf(dt)` averages the last 45 frame times and steps `quality` between 0 (full), 1 (medium), and 2 (low) to hold roughly 40–56 FPS.
4. **Audio.** A lazily created `AudioContext` (`ensureAudio`), with `beep`, `noiseBurst`, and a set of named `sfx.*` helpers. `ensureAudio` is called from input handlers, so audio starts only after a user gesture.
5. **HUD and feedback.** `renderHearts`, `renderBombs`, `showComboPop`, `showStageBanner`, `screenShake`, `flashHit`, `flashBomb`, and `announce` (which writes to the `aria-live` region `#srLive`).
6. **Viewport.** The canvas uses a fixed logical resolution of `BASE_W = 480`, `BASE_H = 720`, scaled by `resize()`. All game coordinates are in this logical space.
7. **Input.** Keyboard (`keys[e.code]`), a touch joystick in `#moveZone`, FIRE and BOMB buttons, and mouse steering and firing on the canvas.
8. **Game state and loop.** `resetGame`, `update(dt)`, `draw()`, and `loop(now)`, which runs under `requestAnimationFrame`. Entities are plain objects in arrays (`bullets`, `enemies`, `powerUps`, and so on), and collisions use `rectsOverlap`.
9. **Weapons and power-ups.** `fireWeapon` (`vulcan` and `laser`, each with levels 1–3), `applyPowerUp`, and `maybeSpawnPowerUp`.
10. **Bosses.** `spawnBoss`, `updateBoss`, `killBoss`, and the `BOSS_PATTERNS` array of bullet-pattern functions (`fireRadial`, `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`, `fireCross`, `fireHomingSeeded`, `fireDoubleRing`).
11. **Flow.** `loseLife`, `endGame` (which writes the best score), `togglePause`, and the `startBtn` handler.

## Gameplay reference (keep in sync with code)

- **Difficulty** (`DIFFICULTIES`): `easy`, `normal` (default), `hard`. Each sets spawn rate, enemy speed, enemy shot chance, lives, starting bombs, and a boss HP multiplier.
- **Stages**: a stage advances every `KILLS_PER_STAGE` (18) kills. Stage bosses come from `BOSSES`, which holds six entries.
- **Enemies**: `grunt` (1 HP), `interceptor` (1 HP, weaves), `tank` (4 HP, appears from stage 2).
- **Power-up drops** (`maybeSpawnPowerUp`, one roll per drop): 6% extra life, 8% bomb, 20% weapon upgrade. The weapon power-up raises the level from 1 to 3; at level 3 it switches weapon (`vulcan` ↔ `laser`) and resets to level 1.
- **Score**: `addScore` multiplies points by `1 + min(9, floor(chain / 5))`, so the multiplier rises every 5 kills up to ×10. `chainTimer` (2.2 s) and `breakChain` reset it.

## Controls

- Mouse: move to steer, click to fire.
- Keyboard: Arrow keys or WASD to move, Space to fire, X or Shift for a bomb, P or Esc to pause.
- Touch: drag left to fly, tap FIRE, tap BOMB.

## Persistence keys

These `localStorage` keys are part of the user's saved state. Do not rename them without a migration:

- `skyfighter.bestScore`
- `skyfighter.muted`
- `skyfighter.difficulty`
- `skyfighter.reducedMotion`

Storage reads and writes are not wrapped in try/catch. If you add more, consider wrapping them so the game still runs when storage is blocked (for example in private browsing).

## Conventions

- **Match the existing style.** The code is vanilla ES2015+ with `const`/`let`, arrow functions, and short section banners in the form `// ---------- Name ----------`. Keep comment density close to what is already there.
- **Keep the single-file structure.** New game logic goes in the existing IIFE. New styles go in the existing `<style>` block. Do not split files unless the user asks.
- **Use the logical coordinate space.** Positions and sizes are in 480×720 units. Do not use raw `window` or CSS pixel sizes for gameplay.
- **Respect reduced motion and accessibility.** Screen shake, flashes, and pulses must go through the existing reduced-motion check. Announce important state changes through `announce()`. Keep `aria` labels on new buttons.
- **Do not add dependencies or CDN scripts** without an explicit request.
- **Keep phone layouts working.** The page is viewport-scaled and includes touch controls, so check changes at phone width.

## Verifying changes

There is no automated test or lint setup. To verify a change:

1. Open `index.html` in a desktop browser and check the console for errors.
2. Play through at least one stage on each difficulty you touched. Check the HUD, the boss bar, game over, and the best-score update.
3. Test touch controls with browser device emulation if you changed input or layout.
4. Confirm that toggling mute, reduced motion, and pause still works and persists across a reload.

Report honestly what you were and were not able to exercise in a browser.

## Git workflow

- Develop on the branch the session specifies. Do not push to other branches without permission.
- Write clear, descriptive commit messages.
- Do not open pull requests unless the user asks for one.
