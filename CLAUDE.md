# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo is **Sky Fighter**, a single-file, dependency-free HTML5 canvas game: a Raiden-style vertical shoot-'em-up with weapons, smart bombs, stage waves, and bosses. Everything lives in `index.html` (inline `<style>` and one inline `<script>`). `README.md` is a placeholder.

There is no build system, package manager, test suite, or linter. Nothing needs to be installed.

## Running and verifying

- Open `index.html` directly in a browser, or serve the folder with any static server (e.g. `python3 -m http.server`).
- Verify changes by playing in a browser. For touch behavior, use a touch device or browser emulation.
- Check at least: desktop keyboard controls, touch joystick and buttons, pause/resume, game over, and the difficulty buttons.

## Code layout (`index.html`)

- **`<style>`**: HUD pills (`#hud`, `.stat`), hearts and bombs, overlay card, vignette/flash effects, touch control layout, and keyframes (`heartBeat`, etc.). Mobile layout uses `clamp()`, `vw`, and `env(safe-area-inset-*)`.
- **HTML body**: `#gameCanvas`, `#hud`, `#overlay` (title, stats, start button, difficulty buttons `.diffBtn`), `#touchControls` (`#moveZone`, `#joystickBase`, `#fireBtn`, `#bombBtn`), `#bossBarWrap`, `#comboPop`, `#stageBanner`, `#flash`, `#vignette`, and an ARIA live region `#srLive`.
- **Script**: a single IIFE with `'use strict'`. Key sections, in file order:
  - DOM references and `STORAGE_*` keys
  - `announce()` (screen reader text) and `trackPerf()`
  - Settings: `DIFFICULTIES` table (spawn timing, enemy and boss tuning per difficulty), persisted best score, mute, difficulty, and reduced-motion flags
  - Audio: `ensureAudio()`, `beep()`, `noiseBurst()`, and the `sfx` helpers (WebAudio, created lazily after a user gesture)
  - UI effects: `renderHearts`, `renderBombs`, `flashHit`, `showComboPop`, `showStageBanner`, `screenShake`
  - Input: `keydown`/`keyup` handlers, virtual joystick, touch buttons, and `canvasPointFromEvent` (mouse/touch on canvas)
  - Game state: `resetGame()`, `spawnEnemy()`, `spawnBoss()`, `maybeSpawnPowerUp()`, `spawnExplosion()`, scoring (`addScore`, `breakChain`), `useBomb()`, `fireWeapon()`
  - `update(dt)`: the main simulation step (spawning, collisions, stage progression)
  - Power-ups (`applyPowerUp`) and boss attack patterns (`fireRadial`, `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`, `fireCross`, `fireHomingSeeded`, `fireDoubleRing`), plus `updateBoss` and `killBoss`
  - Lifecycle: `loseLife()`, `endGame()`, `togglePause()`
  - Rendering: `drawPlane`, `drawBossShip`, `drawClouds`, `draw()`
  - `loop(now)`: `requestAnimationFrame` loop. `dt` is clamped to 0.05s. `update` and `draw` run only when `running && !paused`.

## Conventions

- **Keep it one file.** Add new CSS, markup, and JS to `index.html` unless the user asks for a split. Do not add build tooling or external dependencies.
- **Keep the IIFE scope.** New code should sit inside the existing IIFE and avoid globals.
- **Time-based movement.** Multiply speeds by `dt` in `update` so the game runs the same at any frame rate.
- **Persisted settings** use the `skyfighter.*` localStorage keys. Read and write them through the existing `STORAGE_*` constants. Use `localStorage` only for per-browser preferences such as the best score, mute, difficulty, and reduced motion.
- **Difficulty tuning** belongs in the `DIFFICULTIES` table, not in hard-coded constants inside spawn or update logic.
- **Accessibility**: respect `prefers-reduced-motion` (see `reducedMotion`) when adding shakes, flashes, or other motion effects. Announce important state changes through `announce()`.
- **Mobile**: keep touch controls working with `touch-action: none` and `passive: false` handlers that call `preventDefault()`. Keep layout working at phone widths, with safe-area insets.
- **Audio** starts only after a user gesture (`ensureAudio()`). Keep it that way to avoid autoplay blocks.
- Match the existing style: 2-space indentation, single quotes in JS, `const`/`let`, and small named functions.

## Git

- Develop and push on the branch the session specifies. Do not push to other branches without permission.
- Commit messages are descriptive, imperative, and summarize the gameplay or code change.
