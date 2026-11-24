# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a small static web project. Its only real content is **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs in a browser canvas.

- `index.html` (~1,670 lines): the entire game. Markup, CSS (`<style>`, roughly lines 10–361), HTML body (roughly lines 362–421), and one inline `<script>` IIFE (starts ~line 422) containing all game logic. There is no separate JS or CSS file.
- `README.md`: a placeholder. It has no build or run instructions.
- No `package.json`, bundler, test suite, linter, or CI config exists. Do not invent commands for them.

## Running the game

Open `index.html` directly in a browser, or serve the directory with any static server (for example `python3 -m http.server`). No install or build step is needed.

## Code layout inside `index.html`

The script is wrapped in a single `(function () { 'use strict'; ... })();`. Everything is private to that closure; nothing is exposed on `window`.

Key sections, in order of appearance:

| Area | What it holds |
| --- | --- |
| DOM lookups | `const xxxEl = document.getElementById(...)` block at the top. Element IDs in the markup must match these. |
| Storage keys | `STORAGE_BEST`, `STORAGE_MUTE`, `STORAGE_DIFF`, `STORAGE_REDUCED` (all prefixed `skyfighter.`) used with `localStorage`. |
| Adaptive quality | `trackPerf()` samples frame times and steps `quality` 0–2 when frame rate drops or recovers. |
| Tuning data | `DIFFICULTIES` (spawn rate, enemy speed and fire chance, lives, bombs, boss HP multiplier), `KILLS_PER_STAGE`, `THEMES` (background palettes per stage), `ENEMY_TYPES` (grunt, interceptor, tank), `BOSSES` (six bosses, cycled by stage). |
| Audio | Web Audio helpers: `ensureAudio()`, `beep()`, `noiseBurst()`, and an `sfx` object. Audio is created lazily on the first user gesture. |
| Input | Keyboard (`keys[e.code]`), touch joystick and buttons (`moveZone`, `fireBtn`, `bombBtn`), and mouse on the canvas. |
| Game state | `resetGame()` initialises state. `update(dt)` runs the simulation. `draw()` renders each frame. |
| Weapons and power-ups | `fireWeapon()`, `applyPowerUp()`, `maybeSpawnPowerUp()`, `useBomb()`. |
| Bosses | `spawnBoss()`, `BOSS_PATTERNS` (bullet-pattern functions such as `fireRadial`, `fireSpiralBurst`, `fireHomingSeeded`), `updateBoss()`, `killBoss()`. |
| Flow | `loseLife()`, `endGame()`, `togglePause()`, and `loop(now)` (the `requestAnimationFrame` driver). `loop` clamps `dt` to 50 ms. |

### Conventions

- Game-logic time is in seconds, driven by `dt`. Keep new movement and timers frame-rate independent.
- Entities are plain objects held in arrays. Collision uses `rectsOverlap()`.
- Use the existing `announce()` helper for screen-reader updates instead of writing to `srLive` directly.
- Respect the reduced-motion setting (`reducedMotion`, `body.reduced-motion`) for any new screen shake, flash, or pulse effect.
- Store persistent player settings with the `skyfighter.` key prefix, and wrap `localStorage` access so the game still runs if storage is blocked. The existing code does not guard it, so match its pattern unless you are deliberately improving it.
- The game is designed to run at a logical 480×720 (`BASE_W`, `BASE_H`) and scale through `resize()`. Keep new canvas coordinates in that space.
- Touch controls must work alongside keyboard and mouse. Test layout changes at phone width.

## Making changes

- Make edits in `index.html` unless you are adding a file that is clearly needed. If you split code out into separate files, update the `<script>`/`<link>` tags and this document.
- Keep the single-file, no-dependency design. Do not add frameworks or build tooling without the user asking for it.
- Because nothing is automated, verify changes by loading the page in a browser and playing through the affected path: start, fire, bomb, a boss fight, pause, mute, difficulty change, and game over. Check the console for errors.
- Preserve the existing 2-space indentation and the single-quote style used in the script.

## Git

- Develop on the branch assigned to the session (currently `claude/gifted-tesla-1jxdv9`). Do not push to other branches without permission.
- Do not open a pull request unless the user asks for one.
