# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo contains **Sky Fighter**, a Raiden-style vertical shoot-'em-up game built as a single-file HTML5 canvas app.

- `index.html` — the entire game: inline `<style>`, markup, and one inline `<script>` IIFE. No build step, no dependencies, no package manager.
- `README.md` — minimal. It currently holds only a title and a stray timestamp line, not real documentation.

There is no test suite, linter, or CI configured.

## Running the game

Open `index.html` directly in a browser, or serve the directory with any static server (e.g. `python3 -m http.server`). No install step is needed.

## Code layout (`index.html`)

The script is wrapped in a single `(function () { 'use strict'; ... })()`. Within it, roughly in order:

1. **DOM refs** — `document.getElementById` lookups for HUD, overlay, touch controls, and buttons.
2. **Storage keys** — `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion` (all `localStorage`).
3. **Adaptive quality** — `trackPerf` samples frame times and steps a `quality` level (0 full / 1 medium / 2 low).
4. **Tuning tables** — `DIFFICULTIES` (spawn rates, enemy speed, shoot chance, lives, bombs, boss HP multiplier), `KILLS_PER_STAGE` (18), `THEMES` (per-stage sky palettes), `ENEMY_TYPES` (grunt, interceptor, tank), `BOSSES` (six bosses, cycled by stage), `BOSS_PATTERNS`.
5. **Audio** — `ensureAudio`, `beep`, `noiseBurst`, and the `sfx` table. Audio is created lazily on first user gesture.
6. **HUD and effects** — hearts, bombs, combo pop, stage banner, screen shake, flash.
7. **Input** — keyboard (`keys`), touch joystick and buttons, and mouse steering/firing on the canvas.
8. **Game state** — `resetGame`, `update(dt)`, `draw()`, spawning, collisions, power-ups, bombs, boss logic, `loseLife`, `endGame`, `togglePause`.
9. **Loop** — `loop(now)` runs `update` then `draw` via `requestAnimationFrame`, with `dt` clamped to 0.05s. The start button is wired last.

Game state is kept in plain module-level `let` variables (e.g. `score`, `lives`, `bombCount`, `running`, `paused`, `gameOver`). Rendering is immediate-mode canvas drawing; there is no framework.

## Conventions

- Keep everything in `index.html` unless the user asks for a split. Don't introduce a build tool, bundler, or framework without asking.
- Use plain ES2015+ JavaScript and `const`/`let`. Match the existing style: 2-space indent, single quotes, semicolons.
- New tunable numbers belong in the existing tables (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `THEMES`) rather than as magic numbers in logic.
- Persist new player settings through `localStorage` under the `skyfighter.` prefix, and wrap reads so a missing or malformed value falls back to a default.
- Honor `prefers-reduced-motion` and the in-game reduced-motion toggle when adding screen effects (see `body.reduced-motion` CSS and `reducedMotion` in JS).
- Any new control must work on both desktop (keyboard/mouse) and touch. Update the on-screen `controlsHint` text when controls change.
- Keep the game playable at phone width: the canvas is scaled from a `480×720` base via `resize()`.

## Controls (for reference when editing input code)

- Keyboard: arrows/WASD move, Space fire, X or Shift bomb, P or Esc pause.
- Mouse: move to steer, click to fire.
- Touch: drag left zone to fly, FIRE button, BOMB button.

## Working in this repo

- Make changes with minimal diffs; do not reformat unrelated parts of `index.html`.
- After editing, verify by loading the page in a browser (or the Playwright setup if available) and checking the console for errors. A syntax-only check can be done with `node --check` on the extracted script if needed.
- Do not commit generated files, screenshots, or `node_modules`.
- Commit messages should be short and imperative, describing the gameplay or code change.
