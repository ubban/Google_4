# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Google_4 is a small static web project. Its only app is **Sky Fighter**, a
Raiden-style vertical shoot-'em-up that runs in a single HTML page drawn on a
`<canvas>`.

## Repository layout

- `index.html` (~1,670 lines): the whole game. CSS is in a `<style>` block near
  the top, markup follows, and all game logic is in one inline `<script>`
  (starts around line 422) wrapped in an IIFE.
- `README.md`: a one-line title. Keep it short.
- No build step, package manager, test suite, or linter is configured.

## Running and verifying changes

- Open `index.html` directly in a browser, or serve the directory with any
  static server (for example `python3 -m http.server`).
- There are no automated tests. Verify gameplay changes by playing in a
  browser, and check mobile layout with a narrow viewport.

## Code structure inside the script

- **Config tables** near the top of the script: `DIFFICULTIES` (spawn rates,
  enemy speed and fire chance, lives, bombs, boss HP multiplier),
  `KILLS_PER_STAGE`, `THEMES` (per-stage palettes), `ENEMY_TYPES`, and `BOSSES`.
  Tune balance here rather than in the update logic.
- **Persistence:** `localStorage` keys are prefixed `skyfighter.` (best score,
  mute, difficulty, reduced motion). Reads and writes should tolerate missing or
  blocked storage.
- **Audio:** WebAudio, created lazily by `ensureAudio()` on the first user
  gesture. Don't create an `AudioContext` before a gesture.
- **Input:** keyboard (`keys` map keyed by `e.code`), mouse/pointer via
  `canvasPointFromEvent`, and touch (left-side joystick, FIRE and BOMB buttons).
  Pause is `P` or `Esc`. Bomb is `X` or `Shift`. Fire is `Space` or the FIRE button.
- **Game loop:** `update(dt)` advances state, `draw()` renders it, and `loop(now)`
  drives both through `requestAnimationFrame`. `resetGame()` restores the initial
  state. Keep update and draw separate.
- **Bosses:** `BOSS_PATTERNS` holds attack functions (radial, spread, spiral,
  cross, homing, double ring). `updateBoss` and `killBoss` manage phases.
- **Accessibility:** a reduced-motion setting exists (`screenShake`, flashes, and
  clouds respect it). `announce()` writes to a live region for screen readers.
  Keep both working when adding effects.

## Conventions

- Keep the project dependency-free and self-contained in `index.html` unless the
  user asks otherwise.
- Match the existing style: 2-space indentation, `const`/`let`, single quotes in
  JS, and semicolons.
- Put new tunable numbers in the config tables, not inline in logic.
- The page must work at phone width with touch controls and must not scroll
  horizontally (`overflow: hidden` on `html, body` is intentional).

## Git

- Development branch for this work: `claude/gifted-tesla-nbosdi`. Push there, not
  to other branches, unless the user says otherwise.
- Don't open a pull request unless the user asks for one.
