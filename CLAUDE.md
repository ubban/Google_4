# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo contains **Sky Fighter**, a Raiden-style vertical shoot-em-up game that runs in a browser. It is a single-page HTML5 canvas app:

- `index.html` — the entire game: markup, CSS (`<style>`), and JavaScript (`<script>`) in one file (~58 KB).
- `README.md` — minimal project readme.

There is no build system, package manager, bundler, test suite, or linter configured. Nothing needs to be installed.

## Running the game

Open `index.html` directly in a browser, or serve the directory with any static server, for example:

```bash
python3 -m http.server 8000   # then visit http://localhost:8000/index.html
```

## Code layout inside `index.html`

- **Markup** (`<body>`, ~line 361+): `#container` holds the `<canvas id="gameCanvas">`, HUD (`#hud`: score, best, multiplier, weapon, hearts, bombs), boss bar (`#bossBarWrap`), combo/stage popups, top buttons (`#muteBtn`, `#motionBtn`, `#pauseBtn`), touch controls (`#touchControls`: joystick, FIRE, BOMB), and the start/game-over overlay (`#overlay`, `#overlayCard`, difficulty buttons `.diffBtn`).
- **Script** (`<script>`, ~line 422+): one IIFE with `'use strict'`. Sections, roughly in order:
  - DOM references and `STORAGE_*` keys (localStorage keys are prefixed `skyfighter.`).
  - Config tables: `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES` (per-stage sky palettes), `ENEMY_TYPES`, `BOSSES`.
  - Adaptive quality (`trackPerf`) that steps rendering quality down when frame time is high.
  - Audio: `ensureAudio`, `beep`, `noiseBurst`, and the `sfx` helpers. Audio is synthesized with WebAudio; there are no audio files.
  - HUD helpers (`renderHearts`, `renderBombs`, `showComboPop`, `showStageBanner`, `screenShake`).
  - Input: keyboard (`keys`), touch joystick, on-screen buttons, and mouse on canvas.
  - Game state: `resetGame`, `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`, `addScore`/`breakChain` (kill-chain multiplier), `useBomb`, `fireWeapon`, `applyPowerUp`.
  - Boss attacks: `BOSS_PATTERNS` and the `fire*` pattern functions, driven by `updateBoss`.
  - Game flow: `loseLife`, `killBoss`, `endGame`, `togglePause`.
  - Rendering: `drawPlane`, `drawBossShip`, `drawClouds`, `draw`.
  - Main loop: `loop(now)` calls `trackPerf`, then `update(dt)` and `draw()` while running and not paused. `dt` is clamped to 0.05 s. The loop is started with `requestAnimationFrame`.

Logic is updated in `update(dt)` and drawn in `draw()`. Keep that separation when adding features.

## Controls (for reference when changing input code)

- Move: arrow keys or WASD (`keys[e.code]`, read in `update`), the mouse on the canvas, or the touch joystick.
- Fire: hold Space (touch: FIRE button, or mouse down).
- Bomb: `X` or Shift (touch: BOMB button).
- Pause: `P` or `Escape`, or the pause button.

## Conventions

- Keep everything in `index.html` unless there is a clear reason to split files. If you do split, update this file.
- Use `const`/`let`, the existing 2-space indentation, and single quotes, matching the surrounding code.
- Tuning values (speeds, spawn rates, HP, scores) live in the config tables near the top of the script. Prefer adding to those tables over hard-coding numbers in logic.
- Persist player settings only through the existing `STORAGE_*` keys in localStorage, and wrap any new storage access so the game still works if storage is unavailable.
- Respect the reduced-motion setting (`reducedMotion`) for any new screen shake, flashes, or pulsing effects.
- Keep the mobile layout working: `resize()` scales the 480×720 base canvas to the viewport, so new HUD elements must fit in portrait.
- Pause-safe: new timed behaviour should advance only in `update(dt)` so it freezes while paused.

## Verification

There is no automated test suite. After changing game code:

1. Open `index.html` in a browser (or a local static server) and check the browser console for errors.
2. Play through start, a stage transition, a boss fight, pause/resume, and game over.
3. Check that the best score and settings persist after reload.
4. For layout changes, check a narrow (phone-width) viewport as well as desktop.

## Git

- Development branch for the current work: `claude/gifted-tesla-ig8urq`.
- Write clear, descriptive commit messages in the imperative mood, following the style of existing commits (for example, "Add Sky Fighter: ...").
- Do not open pull requests unless explicitly asked.
