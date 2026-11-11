# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a small static web project. Its only real content is **Sky Fighter**, a Raiden-style vertical shoot-em-up played in the browser on a `<canvas>`.

| Path        | Purpose                                                              |
|-------------|----------------------------------------------------------------------|
| `index.html` | The entire game: markup, CSS (`<style>`), and JavaScript (`<script>`) in one file (~1670 lines). |
| `README.md`  | Minimal placeholder (repo name plus a timestamp line).              |

There is no build step, no package manager, no test suite, and no external dependencies. Open `index.html` in a browser (or serve the directory with any static server) to play.

## Repository layout and conventions

- **Single-file app.** Keep all game code inside `index.html` unless there is a clear reason to split it. Do not introduce a bundler, framework, or npm dependencies without the user asking for them.
- **Structure inside `index.html`:**
  - `<style>` (roughly lines 10–362): layout, HUD, overlay, touch controls, and `body.reduced-motion` overrides.
  - `<body>` markup (from line ~363): `#gameCanvas`, HUD elements (`#score`, `#best`, `#mult`, `#hearts`, `#bombs`, `#weaponName`, `#weaponLvl`, `#bossBarWrap`), the overlay card, touch controls (`#moveZone`, `#fireBtn`, `#bombBtn`), and the icon buttons (`#muteBtn`, `#motionBtn`, `#pauseBtn`).
  - `<script>` (from line ~422): one IIFE with `'use strict'`. All state is closure-scoped; nothing is exposed on `window`.
- **Naming:** `UPPER_SNAKE_CASE` for constants and tuning tables (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `THEMES`, `BOSS_PATTERNS`, `KILLS_PER_STAGE`). `camelCase` for functions and state. `STORAGE_*` constants hold `localStorage` keys.
- **Keep the existing style:** 2-space indentation, single quotes in JS, arrow functions for callbacks, and short comment banners (`// ---------- Section ----------`) to separate sections.

## Game architecture

The loop is a single `requestAnimationFrame` loop, `loop(now)`, which:
1. Computes `dt` in seconds and clamps it to `0.05`.
2. Calls `trackPerf(dt)`, which adapts a `quality` level (0 full, 1 medium, 2 low) based on a 45-frame rolling average.
3. If the game is running and not paused, calls `update(dt)` and then `draw()`.

Key sections, in the order they appear in the script:

- **Settings and persistence:** `DIFFICULTIES` (easy / normal / hard) control spawn rates, enemy speed, enemy fire chance, starting lives, starting bombs, and boss HP multiplier. `KILLS_PER_STAGE` (18) sets stage progression. `THEMES` are six background palettes cycled per stage. Persisted `localStorage` keys are `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, and `skyfighter.reducedMotion`. Reduced motion defaults to the `prefers-reduced-motion` media query until the user toggles it.
- **Enemies:** `ENEMY_TYPES` defines `grunt`, `interceptor` (weaving), and `tank` (4 HP, unlocked at stage 2). `spawnEnemy()` creates them.
- **Bosses:** `BOSSES` is a list of six bosses (VANGUARD through TITAN), each with colors and an `hpMul`. `spawnBoss()` / `updateBoss()` / `killBoss()` manage them. `BOSS_PATTERNS` holds bullet-pattern functions (`fireRadial`, `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`, `fireCross`, `fireHomingSeeded`, `fireDoubleRing`).
- **Player and weapons:** `fireWeapon()`, `applyPowerUp()` (weapon levels, `weaponLabel()`), `useBomb()`, `loseLife()`, and `resetGame()`.
- **Scoring:** `addScore()` applies a combo multiplier. `breakChain()` resets it.
- **Audio:** `ensureAudio()` lazily creates the `AudioContext` (it must be called from a user gesture). `beep()` and `noiseBurst()` synthesize all sound effects. No audio files are loaded.
- **Rendering:** `draw()` renders everything to the canvas. `drawPlane()`, `drawBossShip()`, and `drawClouds()` are the drawing helpers. `resize()` scales a fixed `BASE_W × BASE_H` (480 × 720) logical canvas to the window.
- **Input:** Keyboard (`keys[e.code]`; Space fires, arrows/WASD move, `KeyX` / Shift drops a bomb, `KeyP` / Escape pauses). Pointer and touch use `canvasPointFromEvent()` and a virtual joystick (`joystickStart` / `joystickMove` / `joystickEnd`).
- **Accessibility:** `announce()` writes to the `#srLive` ARIA live region. `reducedMotion` gates screen shake, flashes, and pulses through the `body.reduced-motion` class.

## Working on this codebase

- **Read before editing.** `index.html` is large. Search for the section you need with `Grep` (for example, `function spawnEnemy` or `const BOSSES`) instead of reading the whole file.
- **Tune gameplay in the tables.** Most balance changes (speeds, spawn rates, HP, scores, boss patterns) are data edits in `DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, or `BOSS_PATTERNS`. Prefer those over changing logic.
- **Preserve the performance guard.** Frame-time work should stay cheap. If you add per-frame allocations or heavy drawing, consider how it interacts with `quality`.
- **Preserve accessibility hooks.** When adding new visual effects (flashes, shake, pulsing), gate them on `reducedMotion` and add a matching `body.reduced-motion` rule in CSS.
- **Keep the HUD in sync.** If you add a new state the player should see, add an element to the HUD markup and update it from the game logic.
- **Audio needs a user gesture.** Do not start audio from load-time code.

## Verification

There is no automated test suite or linter. To verify a change:
1. Open `index.html` in a browser (or a headless Chromium with Playwright) and confirm there are no console errors on load.
2. Play through the affected path: start a game, fire, use a bomb, pause and resume, and reach the relevant stage or boss if the change touches them.
3. Check both desktop keyboard controls and the touch controls if the change affects input.
4. Toggle reduced motion and mute to confirm those still work.

## Git and delivery

- Develop on the branch assigned for the session (currently `claude/gifted-tesla-q45o5q`). Do not push to other branches without explicit permission.
- Write clear, descriptive commit messages that explain the gameplay or code change.
- Do not open a pull request unless the user explicitly asks for one.
