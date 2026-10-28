# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Google_4 is a small static web project containing **Sky Fighter**, a Raiden-style
vertical shoot-'em-up played in the browser on a `<canvas>`.

- `index.html` — the entire game: markup, inline CSS, and an inline JavaScript
  module wrapped in an IIFE (`(function () { 'use strict'; ... })();`).
- `README.md` — minimal project readme.

There is no build system, package manager, bundler, framework, or test suite.
Do not add one unless the user asks for it.

## Running the game

Open `index.html` directly in a browser, or serve the directory with any static
server (for example `python3 -m http.server` from the repo root) and visit it.
Nothing needs to be installed.

## Code layout inside `index.html`

The file is organized in roughly this order:

1. `<head>` — meta tags (viewport locked for mobile, theme color `#0b1d3a`) and
   a single `<style>` block.
2. `<body>` — HUD elements, overlay screens (title / game over), touch controls,
   and a `<canvas id="gameCanvas">`.
3. One `<script>` block containing the game:
   - **DOM lookups** at the top, all `const`, named `*El` or `*Btn`.
   - **Storage keys** — `STORAGE_*` constants prefixed `skyfighter.`.
   - **Config tables** — `DIFFICULTIES`, `THEMES` (per-stage sky palettes),
     `ENEMY_TYPES`, `BOSSES`, `KILLS_PER_STAGE`, `BASE_W`/`BASE_H`.
   - **Audio** — `ensureAudio`, `beep`, `noiseBurst`, and `EXPLODE_SFX`. Sound is
     synthesized with the Web Audio API; no audio files are loaded.
   - **Input** — keyboard handlers on `window`, plus joystick and button handlers
     for touch.
   - **Game state** — `resetGame()` builds `player`, `bullets`, `enemies`,
     `enemyBullets`, `particles`, `powerUps`, and `boss`.
   - **Simulation** — `update(dt)` advances the world; `applyPowerUp`, `spawnEnemy`,
     `spawnBoss`, `updateBoss`, `killBoss`, `loseLife`, `endGame` handle gameplay
     events.
   - **Boss attacks** — `BOSS_PATTERNS` is an array of bullet-pattern functions
     (`fireRadial`, `fireAimedSpread`, `fireCross`, etc.) that boss phases pick from.
   - **Rendering** — `draw()` and helpers `drawPlane`, `drawBossShip`,
     `drawClouds`. Rendering is immediate-mode canvas drawing; there are no sprite
     assets.
   - **Main loop** — `loop(now)` runs on `requestAnimationFrame`, clamps `dt` to
     0.05 s, and calls `trackPerf`, `update`, and `draw`.

## Conventions

- **Keep it one file.** New game code goes in the existing `<script>` block, and
  new styles go in the existing `<style>` block. Do not split out modules, add
  external dependencies, or add CDN links unless asked.
- **Match the local style.** The code uses 2-space indentation, single quotes for
  strings, `const` by default with `let` only when reassigned, and arrow functions
  for callbacks. Section headers use the form `// ---------- Name ----------`.
- **Tune gameplay through the config tables.** Difficulty numbers belong in
  `DIFFICULTIES`, enemy stats in `ENEMY_TYPES`, boss stats in `BOSSES`, and
  palettes in `THEMES`. Avoid scattering magic numbers through the update logic
  when a table entry fits.
- **Frame-rate independence.** Movement and timers are scaled by `dt` (seconds).
  Preserve that when adding anything that moves or counts down.
- **Persistence goes through `localStorage` with the `skyfighter.` prefix.** Existing
  keys: `bestScore`, `muted`, `difficulty`, `reducedMotion`. Reads and writes are
  not wrapped in try/catch today; if you add new ones, consider whether storage
  might be unavailable (private mode, blocked storage).
- **Accessibility hooks.** Announcements for screen readers go through
  `announce()` and the `#srLive` live region. Respect the reduced-motion setting
  (`reducedMotion` flag and `body.reduced-motion` class) when adding screen shake,
  flashes, or other motion effects.
- **Adaptive quality.** `trackPerf` lowers the `quality` level when frames run
  slow. New expensive effects should check it rather than always drawing at full
  cost.

## Controls (for reference when testing)

| Action | Keyboard | Touch |
| --- | --- | --- |
| Move | Arrow keys or WASD | Joystick (left side) |
| Fire | Space (hold) | Fire button |
| Smart bomb | X or Shift | Bomb button |
| Pause | P or Escape | — |

## Verification

There are no automated tests or linters. To check a change:

1. Open `index.html` in a browser (or serve the repo root).
2. Start a game, confirm the HUD updates, and play through at least one stage
   transition and one boss fight if the change touches combat, bosses, or stages.
3. Check the browser console for errors. The file should load with none.
4. For touch changes, test with device emulation or a touch-capable device.
5. For a quick syntax check of the inline script, extract it and run
   `node --check` on the extracted file.

## Git workflow

- Develop on the branch the task names (currently `claude/gifted-tesla-w95vhk`).
- Write clear, present-tense commit messages describing the change.
- Do not open pull requests unless explicitly asked.
