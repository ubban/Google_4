# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo is a small, dependency-free web game: **Sky Fighter**, a Raiden-style
vertical shoot-'em-up. The entire app is a single file, `index.html`
(inline `<style>` and one inline `<script>`, HTML5 Canvas 2D). There is no
build step, no package manager, no test suite, and no external scripts.

Files:

- `index.html` — the whole game (markup, CSS, and JavaScript).
- `README.md` — minimal project readme.
- `CLAUDE.md` — this file.

## Running it

Open `index.html` directly in a browser, or serve the directory with any static
server (for example `python3 -m http.server` and visit `http://localhost:8000`).
No install step is needed.

## Code layout inside `index.html`

- **Markup** (`<body>`): `#container` wraps the `<canvas id="gameCanvas">`,
  the `#hud` pill (score, best, multiplier, weapon, hearts, bombs), the
  `#overlay` title/game-over card, `#stageBanner`, `#comboPop`, `#vignette`,
  `#flash`, the boss bar, touch controls (`#touchControls`), and a hidden
  `#srLive` region for screen-reader announcements.
- **CSS** (`<style>`): design tokens are plain values; a `body.reduced-motion`
  class disables animations.
- **JavaScript** (one IIFE, `'use strict'`), roughly in this order:
  1. DOM lookups and `STORAGE_*` localStorage keys (`skyfighter.*`).
  2. Adaptive quality (`trackPerf`, `quality` 0–2) based on frame times.
  3. Tuning tables: `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES`,
     `ENEMY_TYPES`, `BOSSES`, `BOSS_PATTERNS`.
  4. Audio (`ensureAudio`, `beep`, `noiseBurst`) — Web Audio, created lazily on
     first user gesture; `muted` is persisted.
  5. Input: keyboard (`keys`, `keydown`/`keyup`), mouse/pointer, and a
     virtual joystick + buttons for touch.
  6. Game state and logic: `resetGame`, `update(dt)`, `spawnEnemy`,
     `spawnBoss`, `fireWeapon`, `useBomb`, `applyPowerUp`, `updateBoss`,
     `loseLife`, `endGame`, `togglePause`.
  7. Rendering: `draw()`, `drawPlane`, `drawBossShip`, `drawClouds`.
  8. Main loop: `loop(now)` clamps `dt` to 50 ms, then calls `update` and
     `draw` while running and not paused; started via `requestAnimationFrame`.

Gameplay controls: mouse/pointer steers and clicks to fire; Arrows/WASD move;
Space fires; X or Shift drops a smart bomb; P or Esc pauses.

## Conventions

- Keep everything in `index.html` unless the user asks for a split. Do not add
  a bundler, framework, or npm dependency without an explicit request.
- Use plain ES2015+ JavaScript that runs in current evergreen browsers. Keep
  code inside the existing IIFE; do not add globals.
- Tune gameplay by editing the constant tables near the top of the script
  (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `THEMES`) rather than hard-coding
  numbers in logic.
- Persisted player settings use `localStorage` with the `skyfighter.` prefix.
  Wrap new storage access so the game still works if storage is unavailable.
- Preserve accessibility hooks: keep `prefers-reduced-motion` handling and the
  `announce()` live-region messages when adding new UI state changes.
- Keep the layout working on phones: the canvas scales to fit via `resize()`
  with a fixed 480×720 logical resolution (`BASE_W`, `BASE_H`). Do not change
  the logical size without checking touch layout and HUD positioning.

## Verifying changes

There are no automated tests or linters. To verify a change:

1. Open `index.html` in a browser (or serve it locally) and check the browser
   console for errors.
2. Play through at least one stage: shooting, a power-up, a bomb, and a boss
   fight, then confirm game over and that the best score persists on reload.
3. For layout or input changes, check a narrow (phone-width) viewport and the
   touch controls, not just desktop.

## Git

- Develop on the branch assigned to the session (currently
  `claude/gifted-tesla-rzohev`). Do not push to other branches without
  permission.
- Write clear, descriptive commit messages and push with `git push -u origin <branch>`.
- Do not open a pull request unless the user explicitly asks for one.
