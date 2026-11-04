# CLAUDE.md

Guidance for Claude Code when working in this repository.

## What this repo is

A small, single-file browser game: **Sky Fighter**, a Raiden-style vertical shoot-'em-up built with HTML5 Canvas and vanilla JavaScript.

- `index.html` — the entire game: CSS (lines 10–359), markup (lines 361–421), and one inline `<script>` (lines 422–1667) wrapped in an IIFE with `'use strict'`.
- `README.md` — placeholder; contains no real project docs.

There is no build step, package manager, bundler, test suite, linter, or CI config. Nothing needs to be installed.

## Running and checking changes

- Open `index.html` directly in a browser, or serve the directory with any static server (e.g. `python3 -m http.server`).
- There are no automated tests. Verify changes by playing the game in a browser: title screen, each difficulty, weapon/power-up pickup, bomb use, stage transition (every `KILLS_PER_STAGE` = 18 kills), boss fight, pause/resume, game over, and mobile touch controls when touching UI.
- For a syntax-only sanity check of the inline script, extract the `<script>` block and run `node --check` on it.

## Code layout (inside the inline script)

Roughly in order of appearance:

1. **DOM refs and constants** — element lookups by ID at the top of the IIFE; `STORAGE_*` keys for `localStorage`.
2. **Config tables** — `DIFFICULTIES` (spawn rates, enemy speed, shoot chance, lives, bombs, boss HP multiplier), `THEMES` (sky palettes, selected by `(stage - 1) % THEMES.length`), `ENEMY_TYPES` (`grunt`, `interceptor`, `tank`), `BOSSES` (six bosses, selected by `(stage - 1) % BOSSES.length`), `BOSS_PATTERNS` (bullet-pattern generators, indexed per boss design).
3. **Audio** — `ensureAudio()`, `beep()`, `noiseBurst()`, and the `sfx` object. Audio is synthesized with WebAudio; there are no audio files.
4. **UI feedback** — `renderHearts`, `renderBombs`, `flashHit`, `flashBomb`, `showComboPop`, `showStageBanner`, `screenShake`, `announce` (screen-reader live region).
5. **Input** — keyboard (`keys` map), touch joystick and fire/bomb buttons, mouse aim on the canvas.
6. **Game state and loop** — `resetGame()`, `update(dt)`, `draw()`, `loop(now)` driven by `requestAnimationFrame`. `update` and `draw` are the two main places gameplay and rendering live.
7. **Adaptive quality** — `trackPerf()` steps a `quality` level (0–2) based on a 45-frame rolling average of frame time.
8. **Persistence** — best score, mute, difficulty, and reduced-motion preference are stored in `localStorage` under `skyfighter.*` keys. Every access is currently unguarded, so see the notes below.

## Conventions

- **Single file.** Keep the game in `index.html` unless the user asks for a split. Don't add a bundler or framework.
- **Plain ES2015+ JavaScript.** Use `const`/`let`, arrow functions, and template strings to match the existing style. No external libraries or CDN links.
- **Keep the IIFE.** New top-level code belongs inside the existing `(function () { ... })()` so nothing leaks onto `window`.
- **Tune gameplay through the config tables** (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `THEMES`, `KILLS_PER_STAGE`) rather than hardcoding numbers in the update/draw logic.
- **Respect reduced motion.** Effects that shake or flash should check `reducedMotion` (see `screenShake`, `flashHit`, `flashBomb`) and scale down, as the existing code does.
- **Accessibility.** Announce important state changes via `announce()` and keep the `aria-*` attributes on controls in sync when you change them.
- **Match the existing visual style** for HUD pills, overlay cards, and canvas drawing helpers (`drawPlane`, `drawBossShip`, `drawClouds`) when adding new art.

## Gotchas

- **Unguarded `localStorage`.** Reads and writes to `localStorage` are not wrapped in try/catch. In browsers where storage is blocked (some private modes, strict privacy settings), this throws during startup. If you touch persistence, wrap it.
- **`dt` is clamped to 0.05 s** in `loop()`. Movement and timers are frame-rate independent only up to that cap; a long stall (e.g. tab switch) is treated as 50 ms.
- **Quality level affects visuals.** `quality` (set by `trackPerf`) disables or reduces `ctx.shadowBlur` and similar effects on slow devices. Test new glow effects at `quality` 2 as well as 0.
- **Pause and game-over state** are controlled by `paused` and the overlay (`togglePause`, `endGame`). Changes to flow should keep the overlay, start button label, and `statsRow` / `newBestEl` visibility consistent.
- **Mobile layout.** The canvas is scaled to `BASE_W × BASE_H` (480×720) by `resize()`. Coordinates for mouse and touch are converted through `canvasPointFromEvent` and the joystick helpers; use those rather than raw `clientX/Y`.

## Git

- Work on the branch you were given; don't push to other branches.
- Keep commits focused. Commit messages should describe the gameplay or UI change, not the file edited.
- Don't open a pull request unless asked.
