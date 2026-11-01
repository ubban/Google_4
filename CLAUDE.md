# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Sky Fighter is a Raiden-style vertical shoot-'em-up that runs entirely in the browser. The whole game is a single self-contained file, `index.html` (inline CSS, inline JavaScript, no external assets). `README.md` is a placeholder with no setup instructions.

There is no build step, no package manager, no bundler, and no test suite. There is no `package.json`, linter config, or CI.

## Running the game

- Open `index.html` directly in a browser, or serve the directory with any static server (for example `python3 -m http.server` from the repo root, then visit `http://localhost:8000`).
- Verify changes by playing them in the browser. There are no automated tests to run.

## File layout

- `index.html`
  - `<style>` (lines ~10–359): all CSS. Colors and layout are inline; HUD, overlay, touch controls, boss bar, and `.reduced-motion` styles live here.
  - `<body>` (lines ~361–421): DOM for the canvas, HUD, boss bar, pause/mute/motion buttons, touch controls (joystick, FIRE, BOMB), the start/game-over overlay, and difficulty buttons.
  - `<script>` (lines ~422–1667): one IIFE (`'use strict'`) containing all game logic.

## Architecture of the script

Everything sits inside a single `(function () { ... })()` closure. Nothing is exported or attached to `window`, so keep new code inside that closure.

Major sections, roughly in file order:

- **DOM refs and storage keys**: `STORAGE_BEST`, `STORAGE_MUTE`, `STORAGE_DIFF`, `STORAGE_REDUCED` (all under `skyfighter.*` in `localStorage`).
- **Accessibility and feedback**: `announce()` writes to the `#srLive` aria-live region. `trackPerf()` adapts a `quality` level (0–2) from a 45-frame moving average of frame time.
- **Audio**: `ensureAudio()` lazily creates the `AudioContext` (must run from a user gesture). `beep()` and `noiseBurst()` synthesize all sound effects; `sfx.*` exposes named effects, including `sfx.uiClick()` and `sfx.powerup()`. There are no audio files.
- **Tuning data**:
  - `DIFFICULTIES` (`easy` / `normal` / `hard`): `spawnBase`, `spawnMin`, `enemySpeed`, `enemyShootChance`, `lives`, `bombs`, `bossHpMul`.
  - `KILLS_PER_STAGE` (18).
  - `THEMES`: six sky palettes, cycled per stage.
  - `ENEMY_TYPES`: `grunt`, `interceptor` (weaving), `tank` (4 HP, stage 2+).
  - `BOSSES`: six bosses (VANGUARD → TITAN), each with `hpMul` and colors. `BOSS_PATTERNS` holds each boss's attack sequence, built from helpers `fireRadial`, `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`, `fireCross`, `fireHomingSeeded`, and `fireDoubleRing`.
- **Canvas**: fixed logical size `BASE_W = 480` × `BASE_H = 720`. `resize()` scales the canvas CSS size to fit the window (capped at 1.6×). All game coordinates are in the 480×720 space.
- **Input**: keyboard (`keys[]`, arrows/WASD, Space fire, X or Shift bomb, P or Esc pause), mouse on the canvas, and touch (joystick in `#moveZone`, FIRE and BOMB buttons).
- **Game state**: `resetGame()` initializes the state. The core loop is `loop(now)` → `update(dt)` → `draw()`, with `dt` clamped to 0.05 s and updates skipped while paused or not running.
- **Gameplay systems**:
  - Weapons: `fireWeapon()` handles `vulcan` and `laser`, with levels 1–3. `applyPowerUp()` handles `life`, `bomb`, and `weapon` pickups; a weapon pickup at level 3 swaps weapon type.
  - Power-up drops: `maybeSpawnPowerUp()` (life 6%, bomb 8%, weapon 20%).
  - Combo and score: `addScore()`, `breakChain()`, `showComboPop()`.
  - Bombs: `useBomb()`, `flashBomb()`, `screenShake()`.
  - Boss: `spawnBoss()`, `updateBoss()`, `killBoss()`, boss bar via `#bossBarFill`.
  - Damage and end: `loseLife()`, `endGame()`, `togglePause()`.
- **Rendering**: `draw()` plus `drawPlane`, `drawBossShip`, `drawClouds`. HUD is DOM-based (`renderHearts`, `renderBombs`, `#score`, `#mult`, etc.), not drawn on the canvas.

## Conventions

- **Vanilla only.** Do not add frameworks, npm dependencies, or a build step unless the user asks. Keep the game as one HTML file unless there is a clear reason to split it.
- **Keep the closure.** New functions and state go inside the existing IIFE. Use `const` for constants and `let` for mutable state, matching the existing style.
- **Tuning belongs in data tables.** Put balance numbers in `DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, and `THEMES` rather than hard-coding them in logic.
- **Respect reduced motion.** Effects that shake, flash, or pulse must check `reducedMotion` (toggled by `#motionBtn`, defaulting to `prefers-reduced-motion`).
- **Announce important state changes** through `announce()` so screen readers get updates (life lost, stage cleared, etc.).
- **Persist only through the existing keys.** Use the `skyfighter.*` `localStorage` keys, and wrap storage access in a way that tolerates it being unavailable.
- **Touch and mouse parity.** Any new control needs both pointer and touch handling, and must not break the `touch-action: none` layout.
- **Mobile layout.** The HUD, overlay, and touch controls must work at phone widths with safe-area insets (`env(safe-area-inset-*)`).

## Verifying changes

Because there is no test suite:

1. Open `index.html` in a browser (or serve the folder) and start a game.
2. Exercise the area you changed: movement, firing, weapon levels, bombs, bosses at the stage transitions, game over and restart, pause, and each difficulty.
3. Check the mobile layout with browser device emulation, including touch controls.
4. Check the browser console for errors.

Syntax-check the inline script if you want a quick sanity pass, for example by extracting the `<script>` block and running `node --check` on it.

## Git workflow

- Develop on the branch assigned for the task (currently `claude/gifted-tesla-yx4l09`). Do not push to other branches without permission.
- Write clear, descriptive commit messages that explain the gameplay or code change.
- Do not open a pull request unless explicitly asked.
