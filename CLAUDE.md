# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo contains **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs as a single self-contained HTML5 canvas page.

- `index.html` — the entire game: markup, inline `<style>`, and one inline `<script>` (an IIFE). No build step, no dependencies, no external assets.
- `README.md` — minimal project readme.

There is no package.json, bundler, test suite, or linter configured. Do not invent build/test commands; if you add tooling, document it here.

## Running the game

Open `index.html` directly in a browser, or serve the directory with any static server (e.g. `python3 -m http.server`) and visit it. Verify changes by playing in a browser, since there are no automated tests.

## Code structure (`index.html`)

- **HTML/CSS (top of file):** HUD pills (`#hud`, `.stat`), overlay card (`#overlay`), canvas (`#gameCanvas`), vignette/flash layers, and touch controls (`#touchControls`). Styles use CSS `clamp()` and `env(safe-area-inset-*)` for phone layout. Keep the page usable at phone width.
- **Config constants (script):** `STORAGE_*` keys, `DIFFICULTIES` (spawn rate, enemy speed/shot chance, lives, bombs, boss HP multiplier), `KILLS_PER_STAGE`, `THEMES`, `ENEMY_TYPES`, `BOSSES`, `BOSS_PATTERNS`. Tune gameplay here first.
- **Audio:** procedural WebAudio via `beep()` / `noiseBurst()`; `ensureAudio()` must run from a user gesture. Mute state is persisted.
- **Input:** keyboard (`keys` map; Space fires, X/Shift bomb, P/Esc pause, arrows/WASD move), mouse/pointer steering, and touch joystick + FIRE/BOMB buttons.
- **Game state:** `resetGame()` initializes state; `update(dt)` advances simulation; `draw()` renders; `loop(now)` drives both via `requestAnimationFrame`, with `dt` clamped to 0.05s.
- **Persistence:** `localStorage` only, under `skyfighter.*` keys (best score, mute, difficulty, reduced motion). Every access should tolerate storage being unavailable.

## Conventions

- Keep everything in `index.html` unless there is a clear reason to split files. The single-file, no-build design is intentional.
- Preserve the `'use strict'` IIFE wrapper; do not add globals.
- Use `const`/`let`, arrow functions, and the existing naming style (camelCase functions, UPPER_SNAKE constants).
- Game logic should be frame-rate independent: scale movement and timers by `dt`.
- Respect the reduced-motion setting when adding screen shake, flashes, or other motion effects.
- Keep touch and keyboard/mouse input paths working together; a new control needs both.
- Accessibility: announce important state changes through the existing `announce()` / `#srLive` live region.

## Working in this repo

- Development branch: `claude/gifted-tesla-njq4xa`. Commit and push there; do not push to other branches without permission.
- Do not open a pull request unless explicitly asked.
- Make small, focused commits with descriptive messages.
- Before finishing a gameplay change, play-test the affected path in a browser (desktop and, if input code changed, touch emulation).
