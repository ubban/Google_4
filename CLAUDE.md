# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo is **Sky Fighter**, a Raiden-style vertical shoot-'em-up built as a
single-file HTML5 canvas game.

- `index.html` — the entire game: inline `<style>`, markup, and one IIFE
  `<script>` (no build step, no dependencies, no external assets).
- `README.md` — minimal placeholder.

There is no package manager, test suite, linter, or build configured. Open
`index.html` in a browser to run it.

## Running and verifying

- Run locally by opening `index.html` directly, or serve the directory with any
  static server (e.g. `python3 -m http.server`) and visit it.
- There are no automated tests. Verify changes by playing the game in a
  browser and checking the affected behavior (controls, HUD, audio, pause, etc.).
- Test both desktop (keyboard/mouse) and touch layouts when touching input,
  HUD, or the overlay, since the canvas scales to the viewport and the touch
  controls are separate DOM elements.

## Code layout (inside `index.html`)

The script is one IIFE with `'use strict'`. Major sections, roughly in order:

1. **DOM references** — `document.getElementById` lookups at the top.
2. **Storage keys** — `skyfighter.*` keys in `localStorage` for best score,
   mute, difficulty, and reduced motion.
3. **Adaptive performance** — `trackPerf` / `quality` (0 full, 1 medium, 2 low)
   based on rolling frame times.
4. **Tuning tables** — `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES` (per-stage
   sky palettes), `ENEMY_TYPES`, `BOSSES`. Balance changes usually belong here.
5. **Audio** — `ensureAudio`, `beep`, `noiseBurst`, and named SFX helpers.
   Audio is created lazily on first user gesture (`ensureAudio`).
6. **HUD and effects** — hearts/bombs rendering, flashes, combo pops, stage
   banner, screen shake.
7. **Input** — keyboard (`keys` map keyed by `e.code`), mouse steering and
   fire, and touch joystick / FIRE / BOMB buttons.
8. **Game state** — `resetGame`, `update(dt)`, `fireWeapon`, `useBomb`,
   `applyPowerUp`, spawning, collision via `rectsOverlap`.
9. **Bosses** — `BOSS_PATTERNS` (bullet patterns such as radial, spread,
   spiral, cross, homing, double ring) driven by `updateBoss`; `killBoss`.
10. **Rendering** — `draw()` and the helpers `drawPlane`, `drawBossShip`,
    `drawClouds`.
11. **Main loop** — `loop(now)` via `requestAnimationFrame`, with `dt` capping
    and pause handling.

## Controls (keep the hint text in sync)

- Move: mouse, Arrow keys, or WASD.
- Fire: Space (held), or hold the mouse button / touch FIRE button.
- Bomb: X or Shift, or the BOMB button.
- Pause: P or Escape.
- The on-screen help text lives in `#controlsHint` (`index.html`). Update it
  whenever a binding changes.

## Conventions

- Keep everything in `index.html` unless there is a clear reason to split it.
  Do not add a build system, bundler, or framework without being asked.
- Stay with vanilla JS and Canvas 2D. Use `const`/`let`, arrow functions, and
  the existing naming style (camelCase functions, UPPER_CASE constant tables).
- Game constants and balance values belong in the tuning tables near the top
  of the script, not inline in update/draw logic.
- Persist player settings only through the existing `skyfighter.*` localStorage
  keys, and wrap any new storage access so the game still works if storage is
  blocked.
- Respect the reduced-motion setting: new screen shake, flashes, or large
  animations should check `reducedMotion`.
- Keep the UI usable at phone width: the layout uses `touch-action: none`, safe
  area insets, and a `viewport-fit=cover` meta tag. Preserve these.

## Git workflow

- Develop on the branch assigned for the session (currently
  `claude/gifted-tesla-wt9unv`). Do not push to other branches without
  permission.
- Write clear commit messages describing the gameplay or code change.
- Do not open a pull request unless explicitly asked.
