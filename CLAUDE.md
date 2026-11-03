# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo contains **Sky Fighter**, a single-file, Raiden-style vertical shoot-'em-up
built as an HTML5 canvas game. There is no build step, no package manager, and no
test suite.

- `index.html` — the entire game: markup, CSS (`<style>`), and JavaScript (`<script>`).
- `README.md` — minimal project title only.

## Running the game

No build or install is needed. Open `index.html` directly in a browser, or serve the
directory with any static server (e.g. `python3 -m http.server`) and load it.

## Code layout (`index.html`)

- **`<style>`** (~lines 10–420): layout and HUD styling. The page is locked to the
  viewport (`overflow: hidden`, `touch-action: none`); the canvas is scaled to fit.
- **Markup** (~lines 380–420): HUD, touch controls (joystick, FIRE, BOMB), the
  pause/mute/motion buttons, and the start/game-over overlay with difficulty buttons.
- **`<script>`** (IIFE starting ~line 423). Key sections, in order:
  - DOM references and `STORAGE_*` keys (`skyfighter.*` in `localStorage`).
  - Config tables: `DIFFICULTIES`, `THEMES` (per-stage sky palettes), `ENEMY_TYPES`,
    `BOSSES`, and `KILLS_PER_STAGE`.
  - Audio: lazily created `AudioContext` (`ensureAudio`) with `beep` / `noiseBurst`.
  - Canvas setup: fixed logical resolution `BASE_W = 480`, `BASE_H = 720`, scaled to fit.
  - Input: keyboard listeners, joystick and touch handlers.
  - Simulation: `update(dt)`, `updateBoss(dt)`, `spawnEnemy`, `spawnBoss`, `spawnExplosion`.
  - Rendering: `draw()`, `drawPlane`, `drawBossShip`, `drawClouds`.
  - Main loop: `loop(now)` drives update and draw each frame.

## Game rules and controls (for reference when editing gameplay)

- **Move:** Arrow keys or WASD, or the on-screen joystick.
- **Fire:** Space (hold), or the FIRE button.
- **Bomb:** X or Shift, or the BOMB button. Limited per difficulty; clears the screen and
  damages bosses.
- **Pause:** P or Escape, or the pause button.
- **Weapons:** Vulcan and Laser (`player.weapon`). Weapon pickups raise `player.level` up to 3;
  the next pickup at level 3 switches weapon and resets the level to 1.
- **Stages:** Reaching `KILLS_PER_STAGE` kills triggers a boss warning (`bossWarned`). Defeating
  the boss (`killBoss`) increments `stage`, which changes the theme and introduces tougher
  enemies. Bosses come from `BOSSES`, cycling through the list.
- **Difficulty:** Easy / Normal / Hard change spawn rate, enemy speed and fire rate,
  starting lives, bomb count, and boss HP multiplier.

## Persistence

Settings and high score are stored in `localStorage` under `skyfighter.*`:
`bestScore`, `muted`, `difficulty`, `reducedMotion`. Reads and writes are not wrapped in
try/catch, so keep that in mind if you add storage calls (they can throw in some
privacy modes).

## Conventions

- Keep the game in a single self-contained `index.html` unless the user asks to split it.
  Do not add external dependencies or CDN scripts without a request.
- Add new tunables to the config tables (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `THEMES`)
  rather than hard-coding magic numbers in the loop.
- Preserve the existing reduced-motion and mobile touch support when changing visuals or
  input: screen shake, flashes, and pulses should respect `reducedMotion`, and new controls
  should work with both keyboard and touch.
- Match the existing style: 2-space indentation, `const`/`let`, single quotes in JS,
  and short comment headers for sections (`// ---------- Section ----------`).
- The `trackPerf` / `quality` mechanism lowers visual quality on slow devices. Keep new
  expensive effects compatible with it.

## Verification

There is no automated test suite or linter. To verify a change:

1. Open `index.html` in a browser (or serve it locally) and check the browser console for errors.
2. Play through at least one stage transition and a boss fight on the affected difficulty.
3. For input or layout changes, test both keyboard and a touch-sized viewport.

## Git

- Develop on the branch specified by the task (currently `claude/gifted-tesla-ine8ew`).
- Write clear, descriptive commit messages. Do not open pull requests unless asked.
