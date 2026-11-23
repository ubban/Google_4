# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a small static web project. Its only app is **Sky Fighter**, a
Raiden-style vertical shoot-'em-up written as a single self-contained HTML5
canvas page.

Files:

- `index.html` (~1670 lines): the entire game. CSS is in a `<style>` block
  (starts around line 10), markup is in the body (canvas `#gameCanvas`, HUD,
  overlay, touch controls), and all game logic is in one inline `<script>`
  (starts around line 422) wrapped in an IIFE with `'use strict'`.
- `README.md`: a one-line title. Keep it short.
- No build system, package manager, test suite, or dependencies.

## Running and testing

- There is no build step. Open `index.html` directly in a browser, or serve the
  directory with any static server (for example `python3 -m http.server`).
- There are no automated tests or linters configured. Verify changes by
  loading the page and playing it. For UI or gameplay changes, use the `run`
  skill or a headless browser (Playwright with the pre-installed Chromium).
- Check the browser console for errors after any change to the script.

## Code structure (inside `index.html`)

The inline script is organized as follows. Keep new code in the matching section.

- **DOM refs**: `const xxxEl = document.getElementById(...)` at the top.
- **Persistence**: `localStorage` keys are prefixed `skyfighter.`
  (`bestScore`, `muted`, `difficulty`, `reducedMotion`). Every read and write
  must be wrapped in try/catch, because storage can be blocked or throw (for
  example in private windows).
- **Tuning tables**: `DIFFICULTIES`, `THEMES`, `ENEMY_TYPES`, `BOSSES`,
  `KILLS_PER_STAGE`. Balance changes belong here, not scattered through logic.
- **Audio**: Web Audio helpers (`ensureAudio`, `beep`, `noiseBurst`). Audio
  starts only after a user gesture.
- **HUD and feedback**: hearts, bombs, combo pops, stage banner, screen shake,
  hit and bomb flashes. Screen-reader text goes through `announce()`.
- **Input**: keyboard (`keys[e.code]`), mouse, and touch joystick plus
  FIRE/BOMB buttons. Keep keyboard, mouse, and touch behavior in step with the
  on-screen `#controlsHint` text.
- **Weapons and bombs**: `fireWeapon`, `useBomb`, power-up handling in
  `applyPowerUp`. Weapon levels run from 1 to 3.
- **Boss patterns**: `fire*` pattern helpers (radial, spread, columns, spiral,
  cross, homing, double ring) driven by `updateBoss`.
- **Game loop**: `update(dt)` advances state, `draw()` renders, and `loop(now)`
  runs them through `requestAnimationFrame`. `dt` is clamped to 0.05 s.
  `trackPerf` adjusts a `quality` level (0 to 2) when frame time drifts, and
  rendering reads that level to reduce effects on slow devices.
- **Pause and state**: `togglePause`, `endGame`, `resetGame`. The game
  starts from the overlay, and P or Esc pauses.

## Conventions

- Keep everything in the single HTML file unless there is a clear reason to
  split it. Do not add a framework, bundler, or npm dependency for this game.
- Use plain ES2015+ JavaScript that runs in current evergreen browsers. Match
  the existing style: 2-space indent, single quotes in JS, `const` and `let`,
  arrow functions for callbacks.
- Respect `prefers-reduced-motion` (`reducedMotion`) for any new screen shake,
  flashes, or motion effects.
- Keep the game playable on mobile. Any new control must work with touch and
  must not depend on hover.
- Do not hard-code secrets, API keys, or external URLs. The page loads no
  external resources today; avoid adding any without discussion.

## Git workflow

- Develop on the branch assigned for the session (currently
  `claude/gifted-tesla-v31jxu`). Do not push to other branches without
  explicit permission.
- Commit messages should state what changed and why, in the imperative mood
  (the existing history uses messages like "Add Sky Fighter: ...").
- Do not create pull requests unless explicitly asked.

## Notes for future changes

- When you change controls, scoring, or difficulty, update the matching
  text in the HUD or `#controlsHint` so the in-game instructions stay accurate.
- When you add a new persistent setting, add a `skyfighter.*` key and wrap it
  in try/catch.
- If the project grows beyond one page, update this file to describe the new
  layout, build steps, and test commands.
