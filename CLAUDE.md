# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up that runs in the browser.

The entire game is a single self-contained file: `index.html`. It holds the HTML markup, the CSS (in a `<style>` block), and the game logic (in a `<script>` block). There is no framework, no bundler, no `package.json`, and no test suite. `README.md` is a placeholder.

## Running and verifying

- Open `index.html` directly in a browser, or serve the directory with any static server (for example `python3 -m http.server`).
- There is no build step, no linter, and no automated test runner configured. Verify changes by playing the game in a browser and checking the console for errors.
- The page targets both desktop (keyboard and mouse) and touch devices. Test changes to input, layout, or HUD at phone-sized viewports as well as desktop.

## Code layout (inside `index.html`)

- **`<style>`** (lines 10–359): dark color scheme, HUD pills, vignette/flash overlays, and touch control styles. Colors are hard-coded, not tokenized.
- **Markup** (roughly lines 363–421): a single `<canvas id="gameCanvas">`, the HUD (`#hud`: score, best, multiplier, weapon, hearts, bombs), overlays for title/pause/game-over, touch controls, and a `#srLive` region for screen-reader announcements.
- **`<script>`** (starts around line 422) is one IIFE (`'use strict'`). Sections, in order:
  - DOM references and `STORAGE_*` keys.
  - Adaptive performance (`trackPerf`): steps a `quality` level (0/1/2) based on average frame time.
  - Tuning tables: `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES` (per-stage sky palettes), `ENEMY_TYPES` (grunt, interceptor, tank), `BOSSES` (six designs, cycled by stage).
  - Audio: a small WebAudio synth (`ensureAudio`, `beep`, `noiseBurst`, and an `sfx` object). Audio is initialized lazily on the first user gesture.
  - Canvas sizing and input: `BASE_W = 480`, `BASE_H = 720`, `resize()`, keyboard handlers, joystick and fire/bomb buttons for touch, and mouse handlers on the canvas.
  - Game state and loop: `resetGame`, `update(dt)`, `draw()`, and `loop(now)` driven by `requestAnimationFrame`.
  - Weapons and combat: `fireWeapon` (vulcan and laser, each with levels 1–3), `useBomb`, boss attack patterns (`BOSS_PATTERNS`, `fire*` helpers), `applyPowerUp` (life, bomb, weapon).
  - Persistence: best score, mute, difficulty, and reduced-motion preference, all in `localStorage`.

## Conventions

- **Keep everything in `index.html` unless there is a clear reason to split.** Do not introduce a build system, package manager, or framework without asking the user first.
- **Use plain ES2015+ JavaScript and the Canvas 2D API.** Match the existing style: 2-space indentation, single quotes in JS, `const`/`let`, arrow functions for callbacks, and `function` declarations for top-level game helpers.
- **Tune gameplay through the existing tables** (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `THEMES`) rather than adding magic numbers inside update/draw code.
- **Time-based movement.** `update(dt)` takes a delta-time argument; new motion should scale with `dt` so it stays frame-rate independent.
- **Preserve accessibility hooks.** Announce important state changes through `announce()` (which writes to `#srLive`), honor `reducedMotion` (it scales down `screenShake`), and keep keyboard controls working alongside touch.
- **Guard browser storage.** Existing `localStorage` reads and writes do not use try/catch. If you add new storage use, wrap it so the game still runs when storage is blocked (private windows, disabled site data).
- **Keep `pointer-events` and `touch-action` correct** on overlays and controls. The page disables scrolling and pinch zoom on purpose.

## Git and delivery

- Develop on the branch the task specifies. Do not push to other branches without explicit permission.
- Commit with clear, descriptive messages that explain the gameplay or code change.
- Do not open a pull request unless the user asks for one.
- Commit `CLAUDE.md` changes like any other change.
