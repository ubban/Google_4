# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo is a small, self-contained browser game: **Sky Fighter**, a Raiden-style vertical shoot-'em-up.

- `index.html` — the entire game: inline CSS (`<style>`, ~lines 10–362), markup and HUD/touch controls (~lines 363–421), and one inline `<script>` IIFE (~lines 422–1667) containing all game logic, rendering, audio, and input.
- `README.md` — a placeholder; it currently contains only the title and a timestamp line.

There is no build step, package manager, bundler, test suite, or linter. Nothing needs to be installed.

## Running and testing

- Open `index.html` directly in a browser, or serve the directory with any static server (e.g. `python3 -m http.server`) and visit it.
- There are no automated tests. Verify changes by playing the game in a browser: start a run, check firing, enemy/boss patterns, power-ups, bombs, pause, and game over, and confirm that the browser console shows no errors.
- Check the layout at phone width as well as desktop, since the canvas is scaled to fit the viewport and touch controls appear on touch devices.

## Architecture notes

The script is a single IIFE with `'use strict'`. Everything is private to that closure, so there are no globals to clean up.

- **DOM refs** are grabbed once at the top of the script (`canvas`, `ctx`, `overlay`, HUD elements, touch buttons). Keep new element lookups in that block.
- **Canvas** is a fixed logical size, `BASE_W × BASE_H` = 480 × 720. `resize()` scales the CSS size to fit the window (capped at 1.6×). Game coordinates always use the 480×720 space; do not size the canvas from the window.
- **Tuning data** lives in top-level `const` tables near the top of the script: `DIFFICULTIES`, `THEMES` (background palettes per stage), `ENEMY_TYPES`, `BOSSES` (names, colors, `hpMul`), and `KILLS_PER_STAGE`. Prefer adding or changing content here over hard-coding numbers in the update logic.
- **Main loop**: `loop(now)` computes a clamped `dt` (max 0.05 s), calls `trackPerf(dt)`, then while `running && !paused` runs `update(dt)` followed by `draw()`. Game logic belongs in `update`, and rendering belongs in `draw`; keep them separate.
- **Adaptive quality**: `trackPerf` samples the last 45 frame times and steps `quality` (0 full → 2 low) when average frame time is too slow, or back up when it is comfortably fast. Visual effects that are expensive should check `quality`.
- **Boss attacks** are pattern functions (`fireRadial`, `fireAimedSpread`, `fireSpiralBurst`, …) registered in `BOSS_PATTERNS`. Add new attacks there.
- **Persistence** uses `localStorage` with `skyfighter.*` keys (best score, mute, difficulty, reduced motion). Every access to `localStorage` should be tolerant of it being unavailable.
- **Audio** is generated with the Web Audio API (`beep`, `noiseBurst`, and the `sfx` object). Audio is initialized lazily by `ensureAudio()`, which is called from user-gesture handlers. Do not start audio on page load; browsers block it.
- **Accessibility**: `announce()` writes to the `srLive` live region, and there is a reduced-motion mode (`body.reduced-motion`) that defaults to the OS `prefers-reduced-motion` setting. Respect `reducedMotion` for new screen shake, flashes, and particle effects.

## Controls (for reference when testing)

- Keyboard: arrow keys or WASD to move, Space to fire, X / Shift for bomb, P / Esc to pause.
- Touch: joystick in the left zone, fire button, and bomb button.
- Mouse: move the pointer over the canvas to steer; mouse button to fire.

## Conventions

- Keep the project as a single self-contained HTML file unless there is a clear reason to split it. Do not add a framework or build tooling without the user asking.
- Match the existing style: two-space indentation, `const`/`let` (no `var`), arrow functions for callbacks, and short section comments in the form `// ---------- Section ----------`.
- Use the existing `sfx.*` helpers for new sounds, and `announce()` for new screen-reader messages.
- Do not introduce external network calls, CDN scripts, or third-party assets. The game is intended to run from a single file.

## Git

- Development branch for this session: `claude/gifted-tesla-duumw5`. Do not push to other branches without explicit permission.
- Do not open a pull request unless the user asks for one.
- Keep commits focused. Describe the gameplay or code change in the message body rather than only the file name.
