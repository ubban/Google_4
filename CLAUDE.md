# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo contains **Sky Fighter**, a Raiden-style vertical shoot-em-up built as a single self-contained HTML5 canvas page.

- `index.html` — the entire game: markup, CSS (`<style>`, ~lines 10–360), and one IIFE-wrapped `<script>` (~lines 422–1667). There is no build step, no bundler, no package manager, and no external dependencies.
- `README.md` — a one-line placeholder. Keep it short; don't turn it into a design doc.

There are no tests, linters, or CI configuration in the repo today.

## Running the game

Open `index.html` directly in a browser, or serve the directory with any static server (for example `python3 -m http.server`). No install step is needed.

## Code layout inside `index.html`

The script is one `(function () { 'use strict'; ... })();` block. Its sections, in order:

1. **DOM references** — `const xxxEl = document.getElementById(...)` at the top. Add new HUD/overlay elements here.
2. **Constants** — `STORAGE_*` keys, `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES` (per-stage sky palettes), `ENEMY_TYPES`, `BOSSES`.
3. **Persistence and settings** — best score, mute, difficulty, and reduced-motion are read from and written to `localStorage` under the `skyfighter.*` keys.
4. **Audio** — a lazily created `AudioContext` (`ensureAudio`), plus `beep` / `noiseBurst` helpers and the `EXPLODE_SFX` map. Audio must be created from a user gesture.
5. **UI helpers** — hearts/bombs rendering, hit flash, combo pop, stage banner, screen shake, `announce` for screen readers.
6. **Layout** — `BASE_W = 480`, `BASE_H = 720`; `resize()` scales the canvas to fit the viewport.
7. **Input** — `keys` map from `keydown`/`keyup` (by `e.code`), touch joystick and buttons, mouse/pointer steering, and firing.
8. **Game state and simulation** — `resetGame`, `spawnEnemy`, `spawnBoss`, `update(dt)`, weapons (`fireWeapon`), bombs (`useBomb`), power-ups (`applyPowerUp`), scoring and kill chain (`addScore`, `breakChain`).
9. **Boss patterns** — `BOSS_PATTERNS` is an array of bullet-pattern functions (`fireRadial`, `fireSpiralBurst`, `fireHomingSeeded`, …); `updateBoss` and `killBoss` drive the fight.
10. **Rendering** — `drawPlane`, `drawBossShip`, `drawClouds`, and `draw()`. Rendering is canvas 2D only.
11. **Main loop** — `loop(now)` clamps `dt` to 50 ms, calls `trackPerf`, then `update` and `draw` while the game is running and not paused.

## Conventions

- **Single file.** Keep everything in `index.html` unless the user asks for a split. Don't add a build system, framework, or CDN dependency without a request.
- **Match the existing style.** Plain ES2015+ JavaScript, `const`/`let`, arrow functions for callbacks, two-space indentation, single quotes, and terse section banners in the form `// ---------- Name ----------`.
- **Data-driven tuning.** Balance numbers live in the constant tables (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `THEMES`). Prefer adding a table entry over hard-coding a new special case.
- **Frame-rate independence.** Movement and timers use `dt` (seconds). Don't introduce per-frame counters for anything that should be time-based.
- **Adaptive quality.** `trackPerf` adjusts `quality` (0–2) from the rolling frame time. If you add expensive effects, respect `quality` and `reducedMotion`.
- **Accessibility.** Respect `reducedMotion` for shakes, flashes, and similar effects. Announce important state changes through `announce()` so screen readers hear them.
- **Persistence keys.** Use the `skyfighter.` prefix for any new `localStorage` key, and declare it alongside the existing `STORAGE_*` constants. Wrap storage reads in a fallback so the game still runs if storage is unavailable.

## Controls (for reference when changing input code)

- Mouse: move to steer, click to fire.
- Keyboard: Arrows or WASD to move, Space to fire, X or Shift to bomb, P or Esc to pause.
- Touch: drag on the left to fly, FIRE and BOMB buttons on the right.

## Verifying changes

Because there are no automated tests, verify changes by loading `index.html` in a browser and playing through the affected path. For gameplay or rendering changes, check at least:

- Start, pause/resume, and game over still work.
- Difficulty selection and the best score persist across a reload.
- Mute and reduced-motion toggles still apply.
- Layout still fits at desktop and phone widths.
- The browser console shows no errors.

If you can't run a browser in the current environment, say so explicitly rather than claiming the change was tested.

## Git

- Develop on the branch the task specifies. Don't push to `main` without explicit permission.
- Write clear, descriptive commit messages in the imperative mood, with a short subject line and a body when the change needs explaining.
- Don't open a pull request unless the user asks for one.
