# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo is **Sky Fighter**, a single-file, browser-based Raiden-style vertical shoot-'em-up. There is no build step, no package manager, and no test suite.

| File | Purpose |
| --- | --- |
| `index.html` | The entire game: markup (`<body>` overlay/HUD), inline `<style>`, and one inline `<script>` IIFE containing all game logic, rendering, audio, and input. |
| `README.md` | Minimal project readme. |

Run it by opening `index.html` in a browser (or serving the directory with any static server). Nothing needs to be installed.

## Architecture of `index.html`

- **Structure:** HTML + CSS + JS are all inline. Keep it that way unless the user asks to split files. Do not add a bundler or dependencies without asking.
- **Canvas:** One `<canvas id="gameCanvas">` drawn with the 2D context. Logical game space is `BASE_W = 480` by `BASE_H = 720`; `resize()` scales it to the viewport.
- **Game loop:** `requestAnimationFrame` → `loop(now)` → `update(dt)` then `draw()`. `dt` is clamped to 0.05 s. Logic and rendering are both skipped while paused or not running.
- **Configuration tables** (top of the script, near the constants): `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES` (per-stage sky palettes), `ENEMY_TYPES`, `BOSSES`, and `BOSS_PATTERNS`. Prefer adding content by extending these tables rather than adding special cases in the logic.
- **Entities:** The player (`player`), enemies, bullets, power-ups, and explosions are plain objects managed in arrays and updated in `update()`. Bosses are handled by `updateBoss` / `killBoss`.
- **Audio:** Web Audio via a lazily created `audioCtx` (`ensureAudio()`), with helpers `beep()` and `noiseBurst()`. Browsers require a user gesture first, so audio is initialized on the first key press, click, or touch.
- **Persistence:** `localStorage` keys are prefixed `skyfighter.` (`bestScore`, `muted`, `difficulty`, `reducedMotion`). Always wrap storage access in a way that tolerates it being unavailable (private mode, blocked storage) if you change it.
- **Accessibility:** Respect `prefers-reduced-motion` (the `reducedMotion` flag and `body.reduced-motion` class). Announcements go through the `srLive` live region via `announce()`. Keep new UI keyboard- and screen-reader-friendly.

## Controls (for testing)

- **Move:** Arrow keys or WASD. On touch devices, a joystick in `moveZone`.
- **Fire:** Space (hold). Touch: fire button.
- **Bomb:** X or Shift. Touch: bomb button.
- **Pause:** P or Escape.
- **Weapon swap:** Handled in `applyPowerUp()`; the weapon toggles between `vulcan` and `laser`.

## Conventions

- Keep the code vanilla JavaScript (ES2015+ is already in use: `const`/`let`, arrow functions, template literals). No frameworks, no transpilation.
- Keep everything inside the existing IIFE; do not add new globals.
- Match the existing style: 2-space indentation, single quotes in JS, `'use strict'` in effect.
- Game-balance numbers live in the config tables. When tuning, change the table values and note the intent in the commit message.
- Canvas drawing should stay within the logical `480×720` coordinate space; use the existing helpers (`drawPlane`, `drawBossShip`, `drawClouds`) where they fit.

## Verifying changes

There are no automated tests or linters configured. Verify by:

1. Opening `index.html` in a browser (or using the `run` skill / a headless browser) and playing through a stage.
2. Checking the browser console for errors.
3. Exercising the area you changed: at least one full stage transition for logic changes, a boss fight for boss changes, and a mobile-width viewport for layout or touch changes.
4. Checking the reduced-motion and mute toggles if you touched effects or audio.

If you add a test harness or tooling, document its commands here.

## Git workflow

- Develop on the branch designated for the session (currently `claude/gifted-tesla-khabc6`). Do not push to other branches without explicit permission.
- Write commit messages that say what changed and why, in the imperative mood.
- Do not open a pull request unless the user asks for one.
- Do not commit secrets or local config files (for example `.env` or `.claude/settings.local.json`).

## Things to avoid

- Do not split `index.html` into multiple files, add a build system, or add dependencies unless asked.
- Do not change the `localStorage` key names; existing players' saved best scores and settings depend on them.
- Do not remove the reduced-motion or audio-unlock handling.
