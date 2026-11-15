# CLAUDE.md

Guidance for Claude Code when working in this repository.

## What this repo is

`Google_4` is a small, dependency-free static project. Its only shipped app is **Sky Fighter**, a Raiden-style vertical shoot-em-up built as a single HTML5 canvas page.

## Layout

| Path | Purpose |
| --- | --- |
| `index.html` | The entire game: inline `<style>` (~lines 10–361), the `<div id="container">` game shell and HUD/overlay markup (~lines 362–421), and one inline `<script>` IIFE (~line 422 to end) holding all game logic. |
| `README.md` | One-line title only. Keep it short. |

There is no build step, package manager, bundler, test suite, or linter configured. Do not add one unless asked.

## Running it

Open `index.html` directly in a browser, or serve the folder with any static server (e.g. `python3 -m http.server`). No install step is needed.

## Architecture of `index.html`

- **Rendering:** a single `<canvas id="gameCanvas">` drawn with the 2D context. The logical play field is `BASE_W = 480` × `BASE_H = 720`; `resize()` scales the canvas to fit the window. Mouse coordinates are converted with `canvasPointFromEvent`, so keep logic in base-resolution units.
- **Loop:** `requestAnimationFrame(loop)` → `update(dt)` → `draw()`. Pausing is handled by `togglePause()`.
- **Tunables (top of script):** `DIFFICULTIES` (spawn rates, lives, bombs, boss HP multiplier), `KILLS_PER_STAGE`, `THEMES` (per-stage sky palettes), `ENEMY_TYPES`, `BOSSES`, and `BOSS_PATTERNS` (bullet-pattern functions such as `fireRadial`, `fireCross`, `fireSpiralBurst`). Prefer editing these tables over hard-coding numbers in logic.
- **Persistence:** `localStorage` only, with keys prefixed `skyfighter.` (`bestScore`, `muted`, `difficulty`, `reducedMotion`). Every read/write is a plain `localStorage` call; keep them tolerant of missing values.
- **Audio:** Web Audio API, created lazily by `ensureAudio()` on first user gesture (browsers block autoplay). Sounds are procedural (`beep`, `noiseBurst`); there are no audio files.
- **Input:** keyboard (arrows/WASD move, Space fire, X/Shift bomb, P/Esc pause), mouse (move to steer, click to fire), and touch (`#moveZone` joystick, `#fireBtn`, `#bombBtn`). Any new control must work on touch as well as desktop.
- **Adaptive quality:** `trackPerf()` samples frame times and steps a `quality` level down or up to keep frame rate stable.
- **Accessibility:** `announce()` writes to the `#srLive` live region; a reduced-motion toggle (`#motionBtn`) gates screen shake, hit flashes, and vignette pulse. Respect this flag for any new visual effect that moves or flashes.

## Conventions

- **Single file, vanilla JS.** Keep the game in `index.html`. Do not introduce frameworks, bundlers, or external CDN dependencies without asking.
- **Style:** the existing code is ES2015+ inside an `'use strict'` IIFE, uses `const`/`let`, arrow functions for callbacks, and 2-space indentation. Match the surrounding code.
- **Don't leak globals.** New code belongs inside the existing IIFE.
- **Canvas units:** position, size, and speed values are in base-resolution pixels and per-second `dt` values, not CSS pixels.
- **Commit messages:** the existing history uses short imperative subjects with a descriptive body (see `git log`).

## Verifying changes

There are no automated tests. After changing game logic, open the page in a browser and check:

1. Start, play, pause (P/Esc), and game over all work; the overlay shows score, stage, and best.
2. Difficulty buttons change behaviour, and best score persists across reloads.
3. Mute and reduced-motion toggles persist.
4. Touch controls work at phone width (or via DevTools device emulation).
5. No errors in the console.

For visual or gameplay changes, a headless Playwright run with a screenshot is a reasonable check.

## Working notes

- Use `git` on the current feature branch; do not push to other branches without permission.
- Keep this file current: when you add a new top-level file, subsystem, or convention, update the relevant section here.
