# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up that runs entirely in the browser.
The whole game is a single self-contained file, `index.html` (~1,670 lines): inline CSS in `<style>`,
markup in `<body>`, and one IIFE in a single `<script>` block. There is no build step, no bundler,
no package manager, no test suite, and no external dependencies. `README.md` is a placeholder.

## Running the game

- Open `index.html` directly in a browser, or serve the directory with any static server
  (e.g. `python3 -m http.server`) and visit it.
- There is no build, lint, or test command. Verify changes by playing the game in a browser
  (desktop and, where relevant, mobile-sized viewport) and checking the browser console for errors.

## Architecture (all in `index.html`)

- **Layout**: `#container` centers a `<canvas id="gameCanvas">` at a logical 480×720 (`BASE_W`/`BASE_H`),
  scaled by `resize()`. HUD (`#hud`), boss bar, combo/stage banners, top buttons, touch controls,
  and the start/pause/game-over `#overlay` are HTML elements layered over the canvas.
- **Game loop**: `requestAnimationFrame` → `loop(now)` → `update(dt)` then `draw()`. `dt` is clamped
  to 0.05s. `trackPerf()` adapts a `quality` level (0–2) from a rolling frame-time average.
- **State**: module-level `let`s inside the IIFE (`score`, `lives`, `stage`, `bombCount`, `player`,
  `enemies`, `enemyBullets`, `running`, `paused`, `gameOver`, …). `resetGame()` initializes them.
- **Tuning data** (top of the script, around lines 485–527): `DIFFICULTIES` (spawn rate, enemy speed,
  shot chance, lives, bombs, boss HP multiplier), `KILLS_PER_STAGE` (18), `THEMES` (sky palettes cycled
  per stage), `ENEMY_TYPES` (grunt, interceptor, tank), `BOSSES` (six boss designs, indexed by
  `(stage - 1) % BOSSES.length`), and `BOSS_PATTERNS` (bullet patterns, ~line 1372).
- **Weapons and power-ups**: `applyPowerUp()` handles `life`, `bomb`, and `weapon` pickups. Weapons are
  `vulcan` and `laser`, each with levels 1–3; extra pickups at max level toggle to the other weapon.
  `fireWeapon()` implements firing; `useBomb()` clears the screen.
- **Audio**: procedural Web Audio API (`beep`, `noiseBurst`, the `sfx` object). No audio files. Gated by
  `ensureAudio()`, which must run from a user gesture.
- **Drawing**: all sprites are drawn with canvas path code (`drawPlane`, `drawBossShip`, `drawClouds`);
  there are no image assets.
- **Persistence**: `localStorage` keys `skyfighter.bestScore`, `skyfighter.muted`,
  `skyfighter.difficulty`, `skyfighter.reducedMotion`. Reads and writes are not wrapped in try/catch,
  so keep that in mind if storage can be blocked.
- **Accessibility**: a live region (`#srLive`) announces lives, stage, and game-over results via
  `announce()`. A reduced-motion toggle disables screen shake, flashes, and pulses; the initial value
  follows `prefers-reduced-motion` unless the user has set it.

## Controls (keep the hint text in sync)

- Keyboard: arrows/WASD move, Space fires, X or Shift bombs, P or Escape pauses.
- Mouse: move to steer, click to fire.
- Touch: drag the left zone to fly (joystick), tap FIRE, tap BOMB.
- The on-screen hint is `#controlsHint` in the overlay markup. Update it whenever bindings change.

## Conventions

- Keep everything in `index.html` unless the user asks for a restructure. Do not introduce a build
  step, framework, or dependency without asking.
- Keep the code style of the surrounding script: `'use strict'` IIFE, `const` for config, 2-space
  indentation, single quotes in JS.
- Constants for balance belong in the tuning blocks near the top of the script, not inline in logic.
- Any new user-facing text should also be considered for the `announce()` live region when it conveys
  game state.
- Preserve mobile support: touch controls use `passive: false` listeners with `preventDefault()` where
  needed, and the layout must work at phone widths.

## Git workflow

- Develop on the branch specified by the session (currently `claude/gifted-tesla-m7hd5q`), commit
  with clear messages, and push with `git push -u origin <branch>`.
- Do not open pull requests unless explicitly asked.
