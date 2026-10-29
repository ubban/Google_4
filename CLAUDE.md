# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo is a single-page browser game, **Sky Fighter**: a Raiden-style vertical shoot-em-up.

- `index.html` — the entire game: markup (`<body>`), CSS (`<style>`, roughly lines 10–359), and JavaScript (`<script>`, from about line 422 to the end) in one file.
- `README.md` — minimal placeholder.

There is no build step, package manager, bundler, test suite, or linter. Do not add one unless asked.

## Running the game

Open `index.html` directly in a browser, or serve the folder with any static server (for example `python3 -m http.server`). No dependencies are needed.

## Code structure (`index.html`)

All game code lives in one IIFE (`(function () { 'use strict'; ... })();`). Everything is closure-scoped; nothing is exported to `window`.

Main sections, in file order:

- **DOM references and constants** — `canvas`/`ctx`, overlay and HUD element lookups, `STORAGE_*` localStorage keys (`skyfighter.*`).
- **Config tables** — `DIFFICULTIES` (easy/normal/hard), `THEMES` (six stage backdrops), `ENEMY_TYPES` (grunt, interceptor, tank), `BOSSES` (six boss designs), `KILLS_PER_STAGE`, `BASE_W`/`BASE_H` (480×720 logical play area).
- **Audio** — `ensureAudio()`, `beep()`, `noiseBurst()`, `EXPLODE_SFX`. Audio is synthesized with WebAudio; there are no audio files. Mute state persists.
- **Adaptive quality** — `trackPerf()` watches the average frame time and steps `quality` (0 full, 1 medium, 2 low) to keep frame rate up.
- **Layout and input** — `resize()`, keyboard handlers, touch joystick (`joystick*`), and mouse/pointer mapping (`canvasPointFromEvent`).
- **Game state** — `resetGame()`, `spawnEnemy()`, `spawnBoss()`, `maybeSpawnPowerUp()`, `spawnExplosion()`, scoring and combo (`addScore`, `breakChain`), bombs (`useBomb`), weapons (`fireWeapon`, `applyPowerUp`).
- **Simulation** — `update(dt)` advances all entities; `updateBoss(dt)` runs boss attack phases from `BOSS_PATTERNS` (radial, spread, spiral, cross, homing, and similar bullet patterns); `killBoss()`, `loseLife()`, `endGame()`.
- **Rendering** — `draw()` and helpers `drawPlane()`, `drawBossShip()`, `drawClouds()`. Drawing uses the 2D canvas context only.
- **Main loop** — `loop(now)` clamps `dt` to 0.05 s, then runs `update` and `draw` unless paused. Start/restart is wired to `#startBtn`.

Keep new code inside the IIFE and follow the existing style: `const`/`let`, arrow functions for callbacks, two-space indentation, and short `// ---------- Section ----------` comment banners.

## Controls

| Action | Keyboard / mouse | Touch |
| --- | --- | --- |
| Move | Arrow keys or WASD; mouse to steer | Drag left zone (joystick) |
| Fire | Space (hold) or left click | Tap FIRE (hold) |
| Bomb | X or Shift | Tap BOMB |
| Pause | P or Esc | — |

The on-screen hint text is in `#controlsHint`. Keep it in sync if controls change.

## Persistence

Settings and high score use `localStorage` with these keys:

- `skyfighter.bestScore`
- `skyfighter.muted`
- `skyfighter.difficulty`
- `skyfighter.reducedMotion`

Reduced motion defaults to the `prefers-reduced-motion` media query until the player sets it. Always wrap storage reads and writes so the game still works if storage is unavailable.

## Accessibility

- Screen-reader announcements go through `announce()` into the `#srLive` live region.
- `body.reduced-motion` hides or disables vignette, combo, stage-banner, and flash effects via CSS. `screenShake()` also scales shake down while reduced motion is on.
- Keep focusable controls as real `<button>` elements with labels.

## Conventions for changes

- Keep changes minimal and local. The file is large, so prefer targeted edits over rewrites.
- Do not introduce external dependencies, CDNs, or build tooling without a request.
- When you change balance values, update the relevant table (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`) rather than hard-coding numbers in logic.
- Verify gameplay changes by opening the page in a browser (or with the pre-installed Playwright/Chromium) and playing a short session. Check the console for errors.
- Commit messages should be short and imperative, for example `Add boss telemetry` or `Fix bomb cooldown`.
