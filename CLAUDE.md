# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a small web project containing **Sky Fighter**, a Raiden-style vertical shoot-'em-up played in the browser. There is no build system, package manifest, bundler, or test suite.

Files:

- `index.html` (~1,670 lines): the entire game. CSS is in a `<style>` block (roughly lines 10–359), markup follows, and the game logic is in one IIFE inside a `<script>` block (starting around line 422). No external assets; the favicon is an inline SVG data URI.
- `README.md`: placeholder only.

## Running the game

No build step. Open `index.html` directly in a browser, or serve the directory with any static server, e.g. `python3 -m http.server 8000` and visit `http://localhost:8000/`.

## Code structure (inside `index.html`)

The script is a single `(function () { 'use strict'; ... })()` closure. Major sections, in source order:

- **DOM refs and constants**: element lookups, `STORAGE_*` localStorage keys (`skyfighter.bestScore`, `.muted`, `.difficulty`, `.reducedMotion`).
- **Tuning tables**:
  - `DIFFICULTIES` (`easy` / `normal` / `hard`): spawn rates, enemy speed, shoot chance, starting lives and bombs, boss HP multiplier.
  - `KILLS_PER_STAGE` (18): kills needed to advance a stage.
  - `THEMES`: six sky palettes that cycle by stage.
  - `ENEMY_TYPES`: `grunt`, `interceptor` (weaving), `tank` (4 HP, appears from stage 2).
  - `BOSSES`: six bosses, selected by stage, each with its own palette and HP multiplier.
- **Adaptive quality** (`trackPerf`): rolling frame-time average drops or raises a `quality` level (0–2) to keep frame rate up.
- **Audio** (`ensureAudio`, `beep`, `noiseBurst`, `sfx`): Web Audio synthesis only, no sound files. Respects the mute toggle.
- **Rendering** (`draw`, `drawPlane`, `drawBossShip`, `drawClouds`): all drawing is immediate-mode canvas 2D.
- **Input**: keyboard (arrows/WASD move, Space fire, X/Shift bomb, P/Esc pause), mouse on canvas, and touch joystick plus fire/bomb buttons shown on touch devices.
- **Game state and logic**: `resetGame`, `update(dt)`, `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`, `applyPowerUp`, `fireWeapon`, `useBomb`, `loseLife`, `endGame`, `togglePause`.
- **Boss attacks**: `BOSS_PATTERNS` (radial, aimed spread, columns, spiral, cross, seeded homing, double ring), driven by `updateBoss`.
- **Main loop**: `loop(now)` calls `update` and `draw` each frame via `requestAnimationFrame`; the loop starts at the bottom of the script.

Power-up types are `life`, `bomb`, and `weapon` (toggles Vulcan/Laser). The base logical resolution is 480×720 (`BASE_W`/`BASE_H`), scaled by `resize()`.

## Conventions

- Keep the game self-contained in `index.html` unless there is a clear reason to split it. Do not introduce a build tool or framework without asking.
- Keep tuning values in the existing constant tables (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `THEMES`) rather than scattering magic numbers.
- Persist player settings only through the existing `STORAGE_*` keys and wrap new localStorage access so the game still works when storage is unavailable.
- Preserve accessibility hooks: the `srLive` live region for announcements, and the reduced-motion toggle (`reducedMotion`), which dampens screen shake (`screenShake`) and skips hit/bomb flashes (`flashHit`, `flashBomb`).
- Preserve touch support: new controls should work with both pointer/keyboard and the touch overlay.

## Verification

There are no automated tests or linters. To verify a change:

1. Open `index.html` in a browser (or serve it locally) and check the console for errors.
2. Play through at least one stage on each affected path (start, pause/resume, bomb, power-up pickup, life loss, game over, and boss fight if boss logic changed).
3. For layout changes, check at phone width and desktop width (the page must not scroll horizontally).

## Git

- Development branch for this repo: `claude/gifted-tesla-x3753c`. Commit and push there; do not push to other branches without explicit permission.
- Do not open a pull request unless explicitly asked.
