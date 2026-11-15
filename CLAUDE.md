# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo is a small static web project. Its only real content is **Sky Fighter**, a Raiden-style vertical shoot-'em-up written as a single-page HTML5 canvas game.

- `index.html` — the entire game: markup (~lines 1–360), CSS (inline `<style>`, ~lines 10–359), and JavaScript (inline `<script>`, ~lines 422–1667). There is no build step, no bundler, and no external dependencies.
- `README.md` — a placeholder one-liner. Not documentation of the game.

## Running and testing

- **Run:** open `index.html` directly in a browser, or serve the directory with any static server (e.g. `python3 -m http.server`).
- **Build / lint / test:** none exist. There is no package manager, test suite, or CI config. Verify changes by loading the game in a browser and playing through the affected path (menu → gameplay → pause → game over, plus any touched feature).
- Prefer a headless Playwright check when a visual or interaction change needs confirming; the game is canvas-driven, so screenshots are the most reliable check.

## Code structure inside `index.html`

All game logic lives in one IIFE inside the inline script. Key areas, roughly in file order:

- **Persistence keys** (`STORAGE_*`): `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion`. Changing a key name orphans players' saved settings, so treat them as a stable contract.
- **Tuning tables:** `DIFFICULTIES` (spawn rates, enemy speed/fire chance, lives, bombs, boss HP multiplier), `KILLS_PER_STAGE`, `THEMES` (per-stage sky palette), `ENEMY_TYPES` (grunt / interceptor / tank), and `BOSSES` (six bosses cycling by stage). Balance changes should usually be made here rather than in logic.
- **Adaptive quality:** `trackPerf` steps `quality` (0 full → 2 low) based on a rolling frame-time average.
- **Audio:** WebAudio synthesis (`beep`, `noiseBurst`, `ensureAudio`). No audio files are loaded. Audio requires a user gesture, so call `ensureAudio()` from input handlers.
- **Input:** keyboard (`keydown`/`keyup` on `e.code`), touch joystick + fire/bomb buttons, and mouse/pointer movement on the canvas. Keyboard: arrows/WASD move, Space fires, X or Shift bombs, P or Escape pauses.
- **Simulation:** `update(dt)` advances entities, collisions, scoring, and stage progression. `updateBoss` and `BOSS_PATTERNS` drive boss attacks; `loseLife`, `killBoss`, and `endGame` handle state transitions.
- **Rendering:** `draw()` renders everything to the canvas each frame; `drawPlane`, `drawBossShip`, and `drawClouds` are the sprite helpers. `loop(now)` is the `requestAnimationFrame` driver with `dt` clamped to 50 ms.
- **DOM HUD:** hearts, bombs, weapon level, combo pop, stage banner, boss bar, and overlay are DOM elements updated from the game loop. `announce()` writes to a live region for screen readers.

## Conventions

- Keep the game in a single file unless there is a clear reason to split it. The no-build, open-and-run property is a feature.
- Keep the existing style: `const`/`let`, 2-space indent, single quotes in JS, and the IIFE scope (do not add new globals).
- Put new tunable numbers in the existing tables near the top of the script, not inline in logic.
- Respect `reducedMotion`: new screen shake, flash, or heavy particle effects should be gated by it.
- Keep touch and keyboard controls in parity when adding an action.
- Preserve accessibility hooks (`aria-*`, the `srLive` announcements) when changing overlays or the HUD.

## Git

- Development branch for this work: `claude/gifted-tesla-fpv9bn`. Do not push to other branches without explicit permission.
- Commit messages in this repo are short imperative summaries with an optional bulleted body (see `git log`).
