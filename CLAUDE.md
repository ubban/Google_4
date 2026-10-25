# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Google_4 is a small, dependency-free browser game, **Sky Fighter**: a Raiden-style vertical shoot-'em-up. The entire game lives in a single file, `index.html` (about 1,670 lines), with inline CSS in `<style>` and inline JavaScript in one IIFE inside `<script>`.

Files:

- `index.html`: the complete game (markup, styles, and logic).
- `README.md`: a minimal placeholder.
- `CLAUDE.md`: this file.

There is no build step, package manager, test suite, or linter. Open `index.html` directly in a browser to play it.

## Running and verifying

- Run locally: open `index.html` in a browser, or serve the directory with any static server (for example `python3 -m http.server`).
- There are no automated tests. Verify changes by playing the game in a browser, checking the console for errors, and exercising the affected path (keyboard, mouse, and touch where relevant).
- For a quick syntax check of the inline script, extract the `<script>` block and run `node --check` on it.

## Code structure (`index.html`)

The script is one `(function () { 'use strict'; ... })();` closure. Everything is private, so there are no globals to worry about.

- **DOM handles** (top of script): `const` references to elements by id (`gameCanvas`, `overlay`, `startBtn`, `hearts`, `bombs`, `srLive`, and so on). Add new element references here.
- **Persistence**: `localStorage` keys are prefixed `skyfighter.` (`bestScore`, `muted`, `difficulty`, `reducedMotion`). Read and write them through the existing `STORAGE_*` constants.
- **Tuning data** (plain objects and arrays near the top): `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES` (per-stage sky palettes), `ENEMY_TYPES`, and `BOSSES`. Balance changes usually belong here, not in the update logic.
- **Audio**: a Web Audio synth (`ensureAudio`, `beep`, `noiseBurst`). Audio is created lazily on the first user gesture, so keep calling `ensureAudio()` from input handlers.
- **Adaptive quality**: `trackPerf` lowers the `quality` level when frames run slow and raises it when they run fast. Respect `quality` when adding expensive visual effects.
- **Game state and loop**: `resetGame`, `update(dt)`, `draw()`, and `loop(now)`. `loop` clamps `dt` to 50 ms and calls `update` and `draw` only while `running && !paused`.
- **Bosses**: `BOSS_PATTERNS` holds the bullet-pattern functions (radial, spread, spiral, cross, homing, and so on). `updateBoss` and `killBoss` drive them.
- **Input**: keyboard (`keys` map, plus `Space` to fire and `X`/`Shift` for bombs), mouse and pointer over the canvas (`canvasPointFromEvent`), and touch (a virtual joystick plus FIRE and BOMB buttons).
- **Accessibility**: `announce()` writes to the `srLive` live region. `reduced-motion` mode is toggled by a body class, and it defaults to the user's `prefers-reduced-motion` setting.

## Conventions

- Keep the game as a single self-contained HTML file unless the user asks for a split. Do not add a build step or external dependencies without a clear reason.
- Follow the existing style: 2-space indentation, single quotes in JavaScript, arrow functions for callbacks, and `// ---------- Section ----------` comment banners to separate sections.
- Put tunable numbers in the data objects at the top of the script rather than hard-coding them in logic.
- Keep the game working with both mouse/keyboard and touch input. Any new control needs both paths.
- Respect reduced motion: effects that shake or flash should check `reducedMotion`.
- Use `textContent` for any text. `innerHTML` appears only in `renderHearts` and `renderBombs`, where the markup is built from constants and numbers. Do not interpolate user-facing or otherwise untrusted strings into `innerHTML`.

## Git

- Development happens on the branch specified by the session. Do not push to other branches without permission.
- Commit only when asked. Write clear, descriptive commit messages.
