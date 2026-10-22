# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a small static web project. Its only real content is **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs in the browser as a single-file HTML5 canvas app.

- `index.html` — the entire game: inline CSS (`<style>`), markup (HUD, touch controls, start overlay), and one inline `<script>` IIFE with all game logic.
- `README.md` — placeholder; currently contains only the repo name and a stray timestamp.

There is no build step, package manager, bundler, test suite, or linter. Nothing needs to be installed.

## Running and verifying

- Open `index.html` directly in a browser, or serve the directory with any static server (e.g. `python3 -m http.server`).
- There are no automated tests. Verify changes by playing the game in a browser: start a run, test keyboard and mouse controls, check the HUD, and confirm that pause, bombs, weapon power-ups, boss fights, and game over all still work.
- For mobile-related changes, check the touch controls (`#touchControls`) at phone width.

## Code layout inside `index.html`

- **Constants and state**: `STORAGE_*` keys, the `DIFFICULTIES` table, and module-level `let` state (score, lives, weapon level, bombs, stage, etc.).
- **Persistence** (`localStorage`, namespaced under `skyfighter.`): `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion`. Every read and write is currently unguarded, so keep new storage calls consistent with that or wrap them in `try/catch` if you add code paths that may run in restricted browsers.
- **Input**: `keydown` handlers for keyboard, mouse/pointer for steering and firing, and `touchstart` handlers on `#moveZone`, `#fireBtn`, and `#bombBtn`.
- **Audio**: a lazily created `AudioContext` (`ensureAudio`). It is created only after a user gesture.
- **Game loop**: `loop(now)` calls `update(dt)` and then `draw()` each frame via `requestAnimationFrame`. Enemy and boss behavior lives in `update` and `updateBoss`; rendering lives in `draw`, `drawPlane`, `drawBossShip`, and `drawClouds`.
- **Accessibility**: `announce()` writes to the `#srLive` live region. The reduced-motion toggle (`#motionBtn`) disables screen shake, the hit flash, and pulse effects. It defaults to the `prefers-reduced-motion` media query until the user changes it.

## Conventions

- Keep everything in `index.html` unless there is a clear reason to split it. The single-file layout is intentional.
- Match the existing style: 2-space indentation, single quotes in JS, `const`/`let` (no `var`), arrow functions for callbacks, and the `'use strict'` IIFE wrapper.
- Use the existing naming: `camelCase` for functions and variables, `SCREAMING_SNAKE_CASE` for constants, and `skyfighter.`-prefixed storage keys.
- Keep comments sparse and only where the logic is not obvious, as the surrounding code does.
- Do not add external dependencies or CDN scripts. The game is self-contained.
- Preserve the mobile and reduced-motion paths when changing visuals or input.

## Git

- Development branch for this work: `claude/gifted-tesla-vmtqmg`.
- Commit messages in this repo are short imperative summaries, with an optional bulleted body describing the change (see the Sky Fighter commit).
- Do not open pull requests unless explicitly asked.
