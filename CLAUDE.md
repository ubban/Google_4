# CLAUDE.md

Guidance for Claude Code when working in this repository.

## What this repo is

A single-page browser game, **Sky Fighter**: a Raiden-style vertical shoot-'em-up. The whole game lives in one file, `index.html` (about 1,670 lines). `README.md` is a stub and contains no project documentation.

Files:

- `index.html` — markup, inline CSS (`<style>`, roughly lines 10–360), and one inline `<script>` IIFE (roughly lines 422–1668) that holds all game logic, rendering, audio, and input.
- `README.md` — placeholder.

## Build, run, test

- No build step, package manager, or dependencies. Do not add a `package.json`, bundler, or framework unless asked.
- Run by opening `index.html` in a browser, or serve the directory with any static server.
- There is no test suite or linter configured. Verify changes by playing the game in a browser. Check the canvas render, keyboard and touch controls, and the audio path after any change to input or sound code.

## Code layout inside `index.html`

The script is one IIFE with `'use strict'`. Keep new code inside it. Do not leak globals.

- **DOM refs** — `document.getElementById` lookups are grouped at the top of the IIFE.
- **Storage keys** — `STORAGE_*` constants (`skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion`). Any new persisted setting goes under the `skyfighter.` prefix.
- **Tuning tables** — `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES` (sky palettes per stage), `ENEMY_TYPES`, `BOSSES`, and `BOSS_PATTERNS`. Balance changes should go here, not into the logic.
- **Audio** — Web Audio via `ensureAudio()`, `beep()`, and `noiseBurst()`, with a `sfx` object. Audio must start from a user gesture; `ensureAudio()` is already called from the start, fire, and bomb handlers.
- **Input** — keyboard (`keys[e.code]`), a touch joystick and fire/bomb buttons, and mouse on the canvas. Each has its own handlers; keep them in sync when adding a control.
- **Game state and loop** — `resetGame()`, `update(dt)`, and `draw()`, driven by `loop(now)` through `requestAnimationFrame`. `dt` is clamped to 0.05 s. `update` and `draw` run only when `running && !paused`.
- **Adaptive quality** — `trackPerf(dt)` lowers the `quality` level when frames run slow and raises it when they run fast. Respect `quality` when adding expensive effects.
- **Boss fights** — `spawnBoss`, `updateBoss`, `killBoss`, and the pattern functions (`fireRadial`, `fireSpiralBurst`, and so on). The boss HP bar is `bossBarWrap` / `bossBarFill`.
- **Accessibility** — `announce()` writes to a `srLive` live region. Reduced motion is honored from `prefers-reduced-motion` and the user's toggle; the `reduced-motion` class on `<body>` is set from it. Screen shake and flashes should be skipped or softened when it is on.

## Conventions

- Match the existing style: 2-space indentation, single quotes in JS, `const` by default, arrow functions for callbacks, and short `// ---------- Section ----------` banners to group code.
- Keep the file self-contained. The game makes no network requests and loads no external fonts or scripts; the favicon is an inline SVG data URI. Keep it that way unless asked.
- The canvas is a fixed logical size (`BASE_W` × `BASE_H` = 480 × 720) scaled by `resize()`. Draw in logical coordinates.
- Preserve the mobile layout: the viewport meta disables zoom, and touch controls must stay usable at phone width.
- Game-over and high-score handling go through `endGame()` and `STORAGE_BEST`. Do not write the best score anywhere else.

## Git

- Develop on the branch assigned for the session (currently `claude/gifted-tesla-upn7x9`). Do not push to other branches without permission.
- Commit messages are short imperative summaries. The existing history uses messages like `Add Sky Fighter: Raiden-style vertical shoot-em-up game`.
- Do not open a pull request unless the user asks for one.
