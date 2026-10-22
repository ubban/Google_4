# CLAUDE.md

Guidance for working in this repository.

## Project overview

Sky Fighter is a Raiden-style vertical shoot-'em-up that runs in the browser. The entire game ships as one file, `index.html`:

- `<style>` (roughly lines 10–360): all CSS, including the HUD, overlays, touch controls, and the light/dark and reduced-motion styles.
- `<body>` markup (roughly lines 361–421): the `<canvas id="gameCanvas">`, HUD elements, overlays, and touch controls.
- `<script>` (roughly lines 422–1669): one IIFE with `'use strict'` that holds all game logic, rendering, audio, and input.

`README.md` is a stub. There is no build step, no package manifest, no test suite, and no linter configuration.

## Running and checking changes

- Open `index.html` directly in a browser, or serve the directory with any static server (for example `python3 -m http.server`).
- There are no automated tests or lint commands. Verify changes by playing the game in a browser. Check the affected path on desktop (keyboard and mouse) and on touch, or with browser device emulation.
- Keep the file a single self-contained page. Do not add a bundler, framework, or external runtime dependency unless the user asks for it.

## Code structure inside the script

The script is organized by section comments. Keep new code in the matching section:

- **Setup and DOM refs**: `canvas`, `ctx`, and one `const` per HUD or control element. Look elements up once here and do not re-query them in hot paths.
- **Storage keys**: `STORAGE_BEST`, `STORAGE_MUTE`, `STORAGE_DIFF`, `STORAGE_REDUCED`, all prefixed `skyfighter.`. The current code calls `localStorage` directly with no `try`/`catch`. Wrap any new storage access in `try`/`catch`, since storage can be blocked in private or embedded contexts.
- **Adaptive performance** (`trackPerf`, `quality` 0–2): averages the last 45 frame times. It lowers effects quality when frames run slow and raises it when they run fast. Gate expensive effects (`shadowBlur`, particle counts, glow) on `quality`.
- **Tunables**:
  - `DIFFICULTIES`: spawn rate, enemy speed, enemy fire chance, starting lives and bombs, and boss HP multiplier for `easy`, `normal`, and `hard`.
  - `KILLS_PER_STAGE` (18): kills needed before a boss warning.
  - `THEMES`: the six sky palettes that cycle by stage.
  - `ENEMY_TYPES` (`grunt`, `interceptor`, `tank`): size, HP, speed, colors, score, and the stage each one first appears in.
  - `BOSSES`: six bosses with name, colors, and HP multiplier.
  Prefer changing these tables over hard-coding numbers in logic.
- **Audio**: a Web Audio synth (`ensureAudio`, `beep`, `noiseBurst`, `panFor`). There are no audio files. Audio starts only after a user gesture through `ensureAudio()`.
- **Canvas layout**: `BASE_W = 480`, `BASE_H = 720`. `resize()` scales the logical playfield to the viewport. Game coordinates use the base size.
- **Input**: keyboard (`keys[e.code]` map), mouse and pointer steering with `canvasPointFromEvent`, and touch joystick plus FIRE and BOMB buttons. Pause is on P or Esc.
- **Game state**: `resetGame()` initializes all mutable state. `update(dt)` runs the simulation and `draw()` renders it. `loop(now)` drives both through `requestAnimationFrame`, with `dt` capped at 0.05 s.
- **Bosses**: `BOSS_PATTERNS` is an array of bullet-pattern functions (radial, aimed spread, columns, spiral, cross, homing, double ring). `updateBoss`, `killBoss`, and the boss HP bar handle the fight.
- **Accessibility**: `announce()` writes to the `srLive` live region. `reducedMotion` (from the `prefers-reduced-motion` media query, with a user override saved to storage) controls screen shake, flashes, and similar effects.

## Conventions

- Keep the existing style: two-space indentation, single quotes in JS, `const` by default, arrow functions for callbacks, and short `// ----------` section banners.
- Scale movement and timers by `dt` so the game plays the same at any frame rate. Some spawn and fire timers use `dt * 60` (frames at 60 fps); follow the nearby code.
- There is no object pooling. Entities live in plain arrays (`bullets`, `enemyBullets`, `enemies`, `powerUps`, `particles`). Remove them with `filter` or `splice` in `update`, matching the code around them.
- Keep UI text and the controls hint (`#controlsHint`) in sync when you change a binding.
- Touch controls are shown only when `isTouchDevice` is true (the `enabled` class on `#touchControls`). Keep touch-only UI behind that check.

## Working in this repo

- Work on the branch the task names. Do not push to other branches without permission.
- Commit with clear messages that describe the gameplay or code change.
- Test in a browser before you commit a gameplay change. Say in the commit or PR what you tested and what you could not test.
