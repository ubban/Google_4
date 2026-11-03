# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo is **Sky Fighter**, a single-file, browser-based, Raiden-style vertical shoot-'em-up. There is no build step, no package manager, and no test suite.

- `index.html` (~1,670 lines): all markup, CSS (`<style>` near the top) and game logic (one inline `<script>` starting around line 422, wrapped in an IIFE with `'use strict'`).
- `README.md`: a one-line placeholder.

To run the game, open `index.html` in a browser. Nothing else is needed.

## Repository layout and conventions

- Keep the game self-contained in `index.html`. Do not add a bundler, framework, or external script/CSS dependency unless the user asks for it.
- Game state lives in `let` variables inside the IIFE (`player`, `bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`, `boss`, `score`, `lives`, `stage`, and so on). Helpers are plain function declarations.
- Tunable data is declared as `const` objects near the top of the script: `DIFFICULTIES`, `THEMES`, `ENEMY_TYPES`, `BOSSES`, plus `KILLS_PER_STAGE` and `BASE_W`/`BASE_H` (480×720 logical canvas). Prefer changing these over hard-coding numbers in logic.
- Persisted settings use `localStorage` keys prefixed `skyfighter.` (`bestScore`, `muted`, `difficulty`, `reducedMotion`). Always keep the prefix and wrap storage access in a way that still works if storage is unavailable.
- Use the existing `beep`/`noiseBurst`/`sfx` helpers for sound, `announce()` for screen-reader text, and `screenShake`/`flashHit` for feedback. Do not add a new audio stack.

## Game systems (for orientation)

- **Difficulty** (`easy`, `normal`, `hard`): sets spawn rate, enemy speed, enemy fire chance, starting lives, starting bombs, and a boss HP multiplier.
- **Enemies**: `grunt`, `interceptor` (fast, weaving), and `tank` (4 HP, unlocked from stage 2). Enemy types are selected by `minStage`.
- **Stages**: every `KILLS_PER_STAGE` kills triggers a boss. Bosses cycle through `BOSSES` by stage and use `(stage - 1) % BOSSES.length`. Stage themes cycle through `THEMES`.
- **Power-ups** (`life`, `bomb`, `weapon`) drop from enemies and drift down the screen.
- **Scoring**: kills award points scaled by a chain multiplier that decays over time (`chain`, `chainTimer`, `multiplier`).
- **Controls**: arrow keys or WASD to move; Space or the fire button to shoot; X, Shift, or the bomb button to use a smart bomb; P or Esc to pause. Touch uses a virtual joystick and buttons; mouse/pointer also aims.
- **Adaptive quality**: `trackPerf` lowers the `quality` level when frame time is high and raises it when frames are fast. Keep new effects cheap, or gate them on `quality`.
- **Reduced motion**: respect `reducedMotion` for shake, flashes, and similar effects.

## Working on this codebase

- Read the relevant section before editing. The file is large, so use targeted searches (for example `grep -n "function spawnBoss"`) rather than reading everything.
- Keep changes minimal and in the style of the surrounding code (2-space indentation, single quotes in JS, `const`/`let`, arrow functions in callbacks).
- Keep the layout working on phones and desktops. The canvas scales to the viewport via `resize()`, so avoid fixed pixel positions outside the logical 480×720 space.
- Preserve accessibility hooks: `aria-*` attributes, the `srLive` live region, and keyboard controls.
- There are no automated tests. Verify changes by opening `index.html` in a browser (a headless Chromium with Playwright works if one is available) and playing through the affected path. Check the console for errors. Say explicitly in your summary when a change has only been checked by reading the code.

## Git workflow

- Develop on the branch the session specifies. Do not push to other branches without permission.
- Write clear, descriptive commit messages in the imperative mood.
- Do not open a pull request unless the user asks for one.
