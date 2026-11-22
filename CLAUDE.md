# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Google_4 is a small static web project. Its only real content is **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs as a single HTML5 canvas page.

- `index.html` (~1,670 lines): the whole game. CSS is in one `<style>` block, markup is in the body, and all game logic is in one IIFE inside a `<script>` block near the end of the file.
- `README.md`: a placeholder title only.
- No build system, no package manager, no tests, no dependencies. Nothing is fetched from a CDN.

## Running and testing

- Open `index.html` directly in a browser, or serve the folder with any static server (for example `python3 -m http.server`).
- There is no automated test suite or linter. Verify changes by playing the game in a browser, including on a narrow/mobile viewport, since touch controls are part of the design.

## Code structure (inside `index.html`)

- **Storage keys**: `skyfighter.*` in `localStorage` for best score, mute, reduced-motion, and difficulty. Every read and write should tolerate storage being unavailable.
- **Difficulty table** (`DIFFICULTIES`): spawn rate, enemy speed, enemy fire chance, starting lives, starting bombs, and boss HP multiplier for `easy`, `normal`, and `hard`. Tune balance here rather than in the logic.
- **Stage progression**: `KILLS_PER_STAGE` kills advance the stage; a boss spawns and is fought with multiple attack patterns (`fire*` helpers such as radial, aimed spread, spiral, cross, homing, double ring).
- **Game loop**: `update(dt)` handles simulation and `draw()` handles rendering. `loop(now)` drives both via `requestAnimationFrame`. Keep the two separate.
- **Audio**: a WebAudio-based `sfx` helper (`beep`, `noiseBurst`, etc.). It is created lazily by `ensureAudio()`, which must run from a user gesture.
- **Input**: keyboard (arrows/WASD, Space fire, X or Shift bomb, P/Esc pause), mouse on the canvas, and touch (joystick zone on the left, FIRE and BOMB buttons on the right).
- **Accessibility**: there is a reduced-motion toggle that disables screen shake, hit flash, and the low-health pulse. Announcements go through the `#srLive` live region.
- **Layout**: the canvas is scaled by `resize()` and kept inside the viewport with `env(safe-area-inset-*)` handling for notched devices. `touch-action: none` is set on the page, so do not remove it without a replacement.

## Conventions

- Keep the game as a single self-contained file unless the user asks to split it. Do not add a build step or external dependencies without asking.
- Keep the existing style: `const`/`let`, the IIFE wrapper with `'use strict'`, and 2-space indentation.
- Keep game-balance numbers in the `DIFFICULTIES` table or named constants at the top of the script, not inline.
- Use the existing `sfx` helpers for new sounds and the existing HUD elements for new status indicators.

## Git and workflow

- Default working branch for this session: `claude/gifted-tesla-obs1xx`. Push there with `git push -u origin <branch>`.
- Do not open pull requests unless explicitly asked.
- Commit messages should describe the change in the imperative mood and end with the attribution lines the session requires.
