# CLAUDE.md

Guidance for Claude Code when working in this repository.

## What this repo is

A small static web game, **Sky Fighter** (a Raiden-style vertical shoot-'em-up), with no build step and no dependencies.

- `index.html` — the entire game: inline `<style>`, markup, and one inline `<script>` wrapped in an IIFE (`'use strict'`).
- `README.md` — a one-line placeholder.

There is no package.json, bundler, test suite, linter, or CI config. Do not add tooling unless asked.

## Running it

Open `index.html` directly in a browser, or serve the folder with any static server (e.g. `python3 -m http.server`). No install step.

## Layout of `index.html`

- **CSS** (top of file): `:root { color-scheme: dark; }`, HUD pills (`#hud .stat`), overlay, touch controls, vignette/flash effects.
- **Markup**: canvas `#gameCanvas`, HUD elements (score, best, mult, weapon, hearts, bombs, boss bar), overlay with `.diffBtn` difficulty buttons and `#startBtn`, icon buttons `#muteBtn`, `#motionBtn`, `#pauseBtn`, and touch controls (`#moveZone` joystick, `#fireBtn`, `#bombBtn`).
- **Script**: a single IIFE. Notable sections, roughly in file order:
  - Config constants: `DIFFICULTIES` (spawn rates, enemy speed/shoot chance, lives, bombs, boss HP multiplier), `KILLS_PER_STAGE` (18), `THEMES` (six stage skies), `ENEMY_TYPES` (grunt, interceptor, tank), `BOSSES` (six bosses, cycled per stage).
  - Persistence keys: `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion` (all via `localStorage`).
  - Audio: Web Audio helpers (`ensureAudio`, `beep`, `noiseBurst`, `sfx.*`), muted via the mute toggle.
  - Adaptive quality: `trackPerf` steps `quality` 0–2 based on average frame time.
  - Input: keyboard, mouse (canvas aim), and touch (joystick + buttons).
  - Game logic: `resetGame`, `spawnEnemy`, `spawnBoss`, `fireWeapon`, `applyPowerUp`, `useBomb`, `update(dt)`, `updateBoss`, `loseLife`, `endGame`, `togglePause`.
  - Boss attack patterns: `BOSS_PATTERNS` and the `fire*` helpers.
  - Rendering: `draw()`, `drawPlane`, `drawBossShip`, `drawClouds`; main loop `loop(now)` driven by `requestAnimationFrame`.

## Conventions

- **Keep it a single file.** New game code goes in `index.html` unless the user asks for a split. Don't introduce a build step or frameworks.
- **Match the existing style**: 2-space indent, single quotes in JS, `const` by default, small section comments with `// ---------- Name ----------`, and everything inside the IIFE (no new globals).
- **Canvas coordinates**: the game runs at a base resolution of `BASE_W = 480` × `BASE_H = 720` and scales via `resize()`. Use base-resolution units in game logic and drawing.
- **Time-based updates**: `update(dt)` takes a delta time; keep movement and timers frame-rate independent.
- **Accessibility**: respect `reducedMotion` (`screenShake` is dampened and `flashHit`/`flashBomb` are skipped when it is on) and keep the `#srLive` announcements (`announce()`) for important state changes.
- **Storage**: wrap new `localStorage` keys in the `skyfighter.*` namespace and declare them alongside the existing `STORAGE_*` constants.
- **Touch support**: any new control needs both keyboard and touch paths. Touch controls only show when `isTouchDevice` is true.

## Testing and verification

There is no automated test suite. To verify a change:

1. Load `index.html` in a browser (a headless Chromium via Playwright works well).
2. Check the console for errors while playing through start, a stage clear, a boss fight, pause/resume, and game over.
3. For gameplay or layout changes, check both desktop and a phone-sized viewport.

## Git

- Development branch: `claude/gifted-tesla-ua2hsm`. Commit and push there; don't push to other branches without permission.
- Don't open a pull request unless explicitly asked.
