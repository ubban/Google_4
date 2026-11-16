# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a small static web project. Its only real content is **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs in a browser canvas.

- `index.html` (~1,670 lines): the entire game. Inline `<style>`, markup, and one inline `<script>` IIFE. No external assets or dependencies.
- `README.md`: a stub with a title line only.

There is no package manager, build step, bundler, linter, or test suite. Nothing needs to be installed.

## Running and testing

- Run: open `index.html` directly in a browser, or serve the directory with any static server (e.g. `python3 -m http.server` from the repo root, then visit `http://localhost:8000`).
- Tests: none exist. Verify changes by playing the game in a browser. For automated checks, Playwright is available in the cloud environment (`require('playwright')`, no install needed).
- Mobile/touch: the touch controls are shown on touch devices. Check any layout or input change at phone width as well as desktop.

## Architecture (inside `index.html`)

All game logic lives in a single IIFE with `'use strict'`. Key sections, roughly in file order:

| Area | Notes |
| --- | --- |
| Constants | `DIFFICULTIES` (spawn rate, enemy speed, lives, bombs, boss HP multiplier), `KILLS_PER_STAGE` (18), `THEMES` (6 sky palettes, one per stage), `ENEMY_TYPES` (grunt, interceptor, tank), `BOSSES` (6 bosses, cycled by stage). |
| Storage keys | `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion`. All persisted via `localStorage`. |
| Audio | Web Audio synthesis (`beep`, `noiseBurst`, `EXPLODE_SFX`). No audio files. `ensureAudio()` must run from a user gesture. |
| Rendering | `draw()` on a 2D canvas; `drawPlane`, `drawBossShip`, `drawClouds`. Logical size is `BASE_W` x `BASE_H` = 480 x 720, scaled by `resize()`. |
| Update loop | `loop(now)` is a `requestAnimationFrame` loop. `dt` is capped at 0.05 s. `update(dt)` advances state; `draw()` renders it. |
| Adaptive quality | `trackPerf()` samples frame times and steps a `quality` level (0-2) up or down to hold ~40-56 fps. |
| Bosses | `BOSS_PATTERNS` are bullet-pattern functions (`fireRadial`, `fireSpiralBurst`, `fireHomingSeeded`, etc.). `updateBoss` and `killBoss` manage phases. |
| Input | Keyboard, pointer (`canvasPointFromEvent`), and on-screen joystick/fire/bomb for touch. |
| Game state | `resetGame()` initialises state; `endGame()`, `loseLife()`, `togglePause()` handle transitions. |

### Controls

- Move: arrow keys or WASD; touch joystick on touch devices.
- Fire: hold Space (or touch fire button).
- Bomb: X or Shift (or touch bomb button).
- Pause: P or Escape.
- On-screen buttons: mute (`muteBtn`), reduced motion (`motionBtn`), pause (`pauseBtn`).

### Accessibility and preferences

- Honours `prefers-reduced-motion` on first load; the user can override it, and the override is stored.
- Screen-reader announcements go through a polite live region (`announce()`, element `srLive`).

## Conventions

- Keep the game as a single self-contained `index.html` unless the user asks for a split. Do not add a build system, framework, or CDN dependency without asking.
- Match the existing style: 2-space indentation, `const`/`let`, arrow functions for callbacks, short helper functions named in camelCase, and `SCREAMING_CASE` for tables of constants.
- Keep new tunable values in the constant tables (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `THEMES`) rather than hard-coding them in logic.
- Preserve the `localStorage` keys above; changing a key resets players' saved settings and best score.
- Wrap any new storage access in try/catch if you add it, since private windows and blocked storage can throw.
- Audio must only start from a user gesture (keydown, click, touchstart), via `ensureAudio()`.

## Git workflow

- Develop on the branch assigned for the session (currently `claude/gifted-tesla-r5n6x4`). Do not push to other branches without permission.
- Do not open a pull request unless the user asks for one.
- Commit only when asked, or as part of a task that calls for it.
