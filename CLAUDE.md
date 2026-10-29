# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo contains **Sky Fighter**, a Raiden-style vertical shoot-em-up built as a
single-page HTML5 canvas game. The entire game lives in `index.html`:

- `<style>` (inline CSS, roughly lines 10-360): layout, HUD, overlay card, touch controls, themes.
- `<body>` (roughly lines 361-421): DOM for the canvas, HUD, boss bar, touch controls, and overlay.
- `<script>` (roughly lines 422-1667): one IIFE with `'use strict'` containing all game logic.

`README.md` is a one-line placeholder.

## Running and testing

- There is no build step, package manager, bundler, linter, or test suite. Do not add one
  unless asked.
- To run: open `index.html` in a browser, or serve the folder, e.g.
  `python3 -m http.server 8000` and visit `http://localhost:8000`.
- Verify gameplay changes by actually playing them in a browser. Syntax-only checks are not
  enough for changes to rendering, input, or balance.

## Architecture

### Canvas and scaling
- Fixed logical resolution `BASE_W x BASE_H` = `480 x 720`. `resize()` scales the CSS size to
  fit the window, capped at 1.6x. Game coordinates always use the 480x720 space.

### Main loop
- `requestAnimationFrame(loop)` calls `update(dt)` then `draw()` while `running && !paused`.
- `dt` is clamped to 0.05s so tab switches do not cause large jumps.
- `trackPerf(dt)` adapts a `quality` level (0 full, 1 medium, 2 low) from a 45-frame rolling
  average. Use `quality` to scale expensive effects rather than adding fixed-cost ones.

### Game data (top of the script, treat as balance config)
- `DIFFICULTIES`: spawn timing, enemy speed, shoot chance, starting lives and bombs, boss HP
  multiplier for `easy`, `normal`, and `hard`.
- `KILLS_PER_STAGE` (18): kills needed before the stage boss warning.
- `THEMES`: six sky palettes, cycled by stage.
- `ENEMY_TYPES`: `grunt`, `interceptor` (weaving), `tank` (4 HP). Each has a `minStage`.
- `BOSSES`: six bosses, each with colors and an `hpMul`. `BOSS_PATTERNS` holds the attack
  patterns (radial, aimed spread, columns, spiral, cross, homing, double ring).

### Input
- Keyboard: arrows or WASD to move; Space or hold to fire; X or Shift to bomb; P or Escape
  to pause.
- Touch: joystick in `moveZone`, fire and bomb buttons, enabled when a touch device is detected.
- Mouse: the plane glides toward the pointer while hovering the canvas. Keyboard and joystick
  take priority when active.

### Persistence
- `localStorage` keys, all prefixed `skyfighter.`: `bestScore`, `muted`, `difficulty`,
  `reducedMotion`. Always wrap storage access so the game still works if storage is
  unavailable (private mode, blocked site data).

### Audio
- Web Audio, created lazily by `ensureAudio()` on the first user gesture. Browsers block audio
  before a gesture, so do not create the `AudioContext` at load time.
- Helpers: `beep()` for tones and `noiseBurst()` for noise. Sound is toggled by `muted`.

### Accessibility
- `prefers-reduced-motion` is read at load and can be overridden with the motion toggle.
  When reduced motion is on, `screenShake()` scales magnitude to 15% and caps duration at
  0.1s, and the hit and bomb flashes are skipped. New visual effects that flash or shake
  should respect `reducedMotion` the same way.
- `announce()` writes status text into the `#srLive` live region.

## Conventions

- Vanilla JavaScript, ES2015+ syntax (`const`/`let`, arrow functions, template strings). No
  dependencies. Keep it that way.
- Keep everything in `index.html` unless the user asks to split it.
- 2-space indentation, single quotes in JS, semicolons.
- Use the existing section-banner comments (`// ---------- Name ----------`) to group code.
  Add a new banner for a new section instead of a freeform comment.
- Name constants in `UPPER_SNAKE_CASE` and config tables as top-level `const` objects.
  Balance numbers belong in these tables, not inline in update logic.
- Match the existing function style: named `function` declarations for game logic, arrow
  functions for event callbacks.
- Comments should explain why, not restate what the code does.

## Git

- Develop on the branch assigned for the task (currently `claude/gifted-tesla-2mqg98`). Do not
  push to `main` without explicit permission.
- Write clear, descriptive commit messages in the imperative mood.
- Do not open a pull request unless the user asks for one.
