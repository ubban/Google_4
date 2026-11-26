# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo is **Sky Fighter**, a single-file browser game: a Raiden-style vertical shoot-'em-up with weapon power-ups, smart bombs, stage bosses, and a combo/score multiplier.

- `index.html` — the entire game (HTML, inline CSS, and one inline `<script>` IIFE, ~1,670 lines). This is the only code file.
- `README.md` — minimal; currently only a title line.
- No package manager, build step, bundler, test suite, or linter is configured. Do not invent commands for these.

## Running the game

Open `index.html` directly in a browser, or serve the directory with any static server (for example `python3 -m http.server`). No install is needed.

## Architecture of `index.html`

- **Markup**: a `#gameCanvas` (480×720 logical size, scaled by `resize()`), a `#hud` (score, best, multiplier, weapon, hearts, bombs), a boss health bar, a `#topBar` (mute, reduced-motion, pause), touch controls (`#touchControls` with joystick, FIRE, BOMB), and an `#overlay` start/game-over card.
- **Script**: everything lives inside one IIFE with `'use strict'`. Nothing is exported and no globals are added.
  - Tuning tables near the top: `DIFFICULTIES` (easy/normal/hard), `KILLS_PER_STAGE`, `THEMES` (per-stage sky palettes), `ENEMY_TYPES` (grunt, interceptor, tank), and `BOSSES` (six bosses, cycled by stage).
  - Boss attack patterns are in `BOSS_PATTERNS` (radial, aimed spread, columns, spiral, cross, homing, double ring).
  - Game loop: `loop(now)` calls `trackPerf`, then `update(dt)` and `draw()` when running and not paused. `dt` is clamped to 50 ms. Adaptive quality (`quality` 0–2) drops detail when frames run slow.
  - Audio is synthesized with the Web Audio API (`beep`, `noiseBurst`, `sfx`). `ensureAudio()` must run from a user gesture.
- **Persistence**: `localStorage` keys are `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, and `skyfighter.reducedMotion`. Every access is a plain call with no try/catch, so keep that in mind if you change storage use.
- **Accessibility**: `announce()` writes to the `#srLive` aria-live region. Reduced motion (user toggle, defaulting to `prefers-reduced-motion`) gates screen shake, flashes, and pulses.

## Conventions

- Keep the game a single self-contained file unless the user asks for a split. Do not add a build step or dependencies without asking.
- Keep the code style already in use: 2-space indent, `const`/`let`, arrow functions for callbacks, and tuning values in the constant tables rather than scattered as magic numbers.
- Gameplay changes should go through the existing tables (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `THEMES`) where possible.
- Controls are documented in the on-screen `#controlsHint` text. Keep it in sync if key or touch bindings change.
- The layout must work at phone width with touch input and no horizontal page scroll (`overflow: hidden` on body is intentional).

## Verifying changes

There are no automated tests. After editing `index.html`:

1. Open it in a browser and confirm there are no console errors.
2. Play a few seconds with keyboard and, if relevant, touch controls.
3. Check the affected feature directly: HUD values, difficulty selection, mute/reduced-motion toggles, pause, bomb, boss fight, or game over and best-score update.

## Git

- Development branch for this session: `claude/gifted-tesla-n51lr3`. Commit and push there.
- Do not open a pull request unless the user asks for one.
