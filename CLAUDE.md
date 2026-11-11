# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo is a single-file browser game, **Sky Fighter**: a Raiden-style vertical shoot-'em-up.

- `index.html` — the entire game: inline CSS (`<style>`), markup, and one inline `<script>` IIFE. No build step, no dependencies, no package manager.
- `README.md` — minimal placeholder.

To run it, open `index.html` in a browser (or serve the directory with any static server). There is no test suite, linter, or CI configured.

## Code layout inside `index.html`

- **Markup** (~lines 362–420): `#container` holds the `<canvas id="gameCanvas">`, HUD (`#hud`), boss bar, stage/combo banners, top-bar buttons (`#muteBtn`, `#motionBtn`, `#pauseBtn`), touch controls (`#touchControls`), and the start/pause overlay (`#overlay`).
- **Config tables** (~lines 485–528): `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES` (per-stage sky palettes), `ENEMY_TYPES`, `BOSSES`.
- **Persistence**: `localStorage` keys `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion`. Wrap any new storage access in try/catch, since storage can be blocked.
- **Audio**: a WebAudio-based `sfx` layer (`ensureAudio`, `beep`, `noiseBurst`). Audio is created lazily on the first user gesture; keep it that way.
- **Game state & loop**: `resetGame()` initialises state; `update(dt)` advances the simulation; `draw()` renders; `loop(now)` drives both via `requestAnimationFrame`, clamping `dt` to 0.05s.
- **Bosses**: `spawnBoss`, `updateBoss`, `killBoss`, and the `BOSS_PATTERNS` array of bullet-pattern functions (`fireRadial`, `fireAimedSpread`, `fireHomingSeeded`, etc.).
- **Input**: keyboard (`keys` map on `keydown`/`keyup`), mouse on the canvas, and touch joystick plus FIRE/BOMB buttons. Touch is detected via `isTouchDevice`.

## Controls (from the in-game hint)

- Mouse: move to steer, click to fire.
- Keyboard: Arrows/WASD move, Space fire, X or Shift bomb, P or Esc pause.
- Touch: drag left side to fly, tap FIRE, tap BOMB.

## Conventions

- Keep everything in `index.html` unless there is a clear reason to split it. Do not introduce a bundler or framework without asking.
- The script runs in `'use strict'` inside an IIFE. Do not leak globals.
- Game balance lives in the config tables and constants near the top of the script (difficulty, spawn rates, enemy HP/score, boss HP multipliers). Prefer editing those over hardcoding numbers in logic.
- Respect the reduced-motion setting: screen shake, flashes, and pulses should go through the existing helpers (`screenShake`, `flashHit`, `flashBomb`) so they honour `reducedMotion`.
- Keep the layout working at phone width and in both mouse and touch modes. Use `touch-action: none` and `user-select: none` on the game surface as the existing code does.
- Accessibility: announcements for screen readers go through `announce()` into `#srLive`.

## Verifying changes

There are no automated tests. For changes to gameplay or rendering, open `index.html` in a browser and play through at least one stage and one boss, checking:

- Start, pause/resume, and game-over flows, including the best-score update.
- Each difficulty level and the mute and reduced-motion toggles.
- Touch layout (use browser device emulation) and keyboard controls.
- The browser console is free of errors.

## Git

- Develop on the branch designated for the session (currently `claude/gifted-tesla-5umu0e`). Do not push to other branches without permission.
- Do not open a pull request unless explicitly asked.
