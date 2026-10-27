# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo is **Sky Fighter**, a single-file, Raiden-style vertical shoot-'em-up that runs in the browser. There is no build step, no package manager, and no test suite.

- `index.html` — the entire game: markup, CSS (`<style>`), and JavaScript (`<script>`) in one file (~1,670 lines).
- `README.md` — a placeholder; it has no run or build instructions.

To play, open `index.html` directly in a browser, or serve the directory with any static server.

## Layout of `index.html`

- **Markup and styles (lines ~1–420):** HUD elements (score, best, multiplier, weapon, hearts, bombs, boss bar), the start/pause overlay, touch controls, and the `#gameCanvas` element.
- **Script (starts at line ~422):** one IIFE with `'use strict'`. Everything is scoped inside it, so nothing leaks to `window`.
  - **Config constants:** `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES` (per-stage sky palettes), `ENEMY_TYPES`, `BOSSES`, and `BOSS_PATTERNS`.
  - **Persistence:** `localStorage` keys are prefixed `skyfighter.` (`bestScore`, `muted`, `difficulty`, `reducedMotion`).
  - **Audio:** WebAudio synth helpers (`beep`, `noiseBurst`). Audio starts only after a user gesture via `ensureAudio()`.
  - **Input:** keyboard (`keys` map keyed by `e.code`), mouse/pointer via `canvasPointFromEvent`, and touch joystick plus fire/bomb buttons.
  - **Simulation:** `update(dt)` advances game state, `updateBoss(dt)` drives boss phases, and `fireWeapon` / `useBomb` / `applyPowerUp` handle player actions.
  - **Rendering:** `draw()` renders everything to the 2D canvas; `drawPlane`, `drawBossShip`, and `drawClouds` are helpers.
  - **Loop:** `loop(now)` is a `requestAnimationFrame` loop. It clamps `dt` to 50 ms and runs `trackPerf`, which adapts a `quality` level (0–2) to frame time.

## Controls

| Action | Keys / input |
| --- | --- |
| Move | Arrow keys or WASD, or touch joystick |
| Fire | Space (held), or touch fire button |
| Bomb | X or Shift, or touch bomb button |
| Pause | P or Escape |

## Conventions

- Keep the game self-contained in `index.html` unless the user asks for a split. Do not add a bundler, framework, or dependencies without a clear reason.
- Match the existing style: 2-space indentation, single quotes in JS, `const` by default, arrow functions for callbacks, and section banners written as `// ---------- Name ----------`.
- Game tuning belongs in the config constants near the top of the script, not inline in logic.
- Any new persisted setting should use the `skyfighter.` key prefix and be wrapped the same way as the existing `localStorage` reads.
- Preserve accessibility hooks: the `srLive` live region (`announce()`), the reduced-motion setting (`reducedMotion`), and the `aria-*` attributes on controls.
- Preserve the mobile layout: the canvas is scaled to fit via `resize()` (base size 480×720), and touch controls appear only on touch devices.

## Working on the game

- There are no automated tests or linters. Verify changes by loading `index.html` in a browser and playing through the affected path (movement, firing, bombs, boss phases, pause, game over, and the best-score save).
- For visual or gameplay changes, check desktop and a narrow mobile viewport.
- Be careful with `update()` and `draw()`: they run every frame, so avoid allocations or heavy work inside them where you can.

## Git

- Develop on the branch the session specifies. Do not push to other branches without permission.
- Do not open a pull request unless the user asks for one.
