# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Google_4 is a small static web project. Its only game code is **Sky Fighter**, a
Raiden-style vertical shoot-em-up in a single HTML file.

| File        | Purpose                                                        |
| ----------- | -------------------------------------------------------------- |
| `index.html` | The whole game: inline CSS (`<style>`), markup, and one IIFE `<script>` |
| `README.md`  | Placeholder title only                                         |

There is no `package.json`, build step, bundler, linter, or test suite. Nothing
needs to be installed. To run the game, open `index.html` in a browser (or serve
the directory with any static server).

## Working on the game

- Keep the game in the single `index.html` file unless the user asks to split it.
- The script is wrapped in an IIFE with `'use strict'`. Do not add globals; keep
  new state and functions inside that closure.
- Match the existing style: 2-space indentation, `const`/`let`, arrow functions for
  callbacks, `// ---------- Section ----------` banners between logical sections,
  and short comments that explain intent rather than restate code.
- Tunable values live in named constants near the top of the script:
  - `DIFFICULTIES` (spawn rates, enemy speed, shot chance, lives, bombs, boss HP multiplier)
  - `KILLS_PER_STAGE`, `THEMES` (per-stage sky palettes), `ENEMY_TYPES`, `BOSSES`
  - `BOSS_PATTERNS` (boss bullet patterns)
  Prefer changing these over hard-coding numbers in update/draw logic.
- Persisted settings use `localStorage` keys under the `skyfighter.` prefix
  (`bestScore`, `muted`, `difficulty`, `reducedMotion`). Keep that prefix for new keys.
  Wrap storage access so the game still works if storage is unavailable.

### Architecture of the script

- **Setup:** DOM lookups, storage keys, and settings loaded from `localStorage`.
- **Adaptive quality:** `trackPerf` lowers quality when frame time is high.
- **Audio:** WebAudio helpers (`beep`, `noiseBurst`). Audio starts only after a user
  gesture via `ensureAudio()`.
- **Input:** keyboard (`keys` map), mouse/pointer steering, and touch joystick/buttons.
- **Simulation:** `update(dt)` advances enemies, bullets, power-ups, collisions, and
  stage progression. `dt` is clamped to 50 ms in the loop.
- **Rendering:** `draw()` paints everything each frame on the canvas. `resize()` keeps
  the fixed `BASE_W` × `BASE_H` (480 × 720) play field scaled to the window.
- **Loop:** `loop(now)` runs on `requestAnimationFrame` and only updates/draws while
  `running && !paused`.

### Controls (keep the in-game hint in sync)

- Mouse: move to steer, click to fire.
- Keyboard: arrows or WASD to move, Space to fire, X or Shift to bomb, P or Esc to pause.
- Touch: drag left side to fly, FIRE and BOMB buttons.
- The hint text is in `#controlsHint` in the markup. Update it if controls change.

## Accessibility and UX conventions

- Announcements for screen readers go through `announce()` and the `#srLive` region.
- Honor `prefers-reduced-motion` (the `reducedMotion` setting and the `reduced-motion`
  body class). New screen-shake or flash effects should respect it.
- Keep the layout usable on phones; touch controls are shown on touch devices.

## Verification

Because there is no tooling, verify changes by:

1. Opening `index.html` in a browser (or a headless browser) and checking the console
   for errors.
2. Playing through the affected path: start, fire, bomb, pause/resume, stage change,
   boss fight, and game over, as relevant to the change.
3. Testing at desktop and phone widths when touching layout or controls.

Do not add a test framework or build tooling unless the user asks for it.

## Git

- Develop on the branch the task specifies. Do not push to other branches without
  permission.
- Write commit messages that describe the change and its reason, in the imperative mood.
- Do not open pull requests unless the user asks.
