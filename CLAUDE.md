# CLAUDE.md

Guidance for Claude Code when working in this repository.

## What this repo is

A small static web project. Its only real content is **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs in a browser canvas.

- `index.html` — the entire game: markup, CSS (`<style>`, ~lines 10–362), and one inline IIFE script (`<script>`, starts ~line 422). There is no build step, bundler, package manager, or test suite.
- `README.md` — a placeholder with a timestamp line; not documentation of the game.

## Running and verifying

- Open `index.html` directly in a browser, or serve the directory with any static server (e.g. `python3 -m http.server` from the repo root) and visit it.
- There are no automated tests or linters configured. Verify changes by playing the game in a browser (desktop keyboard/mouse and, if relevant, touch emulation).
- Do not add a build system, framework, or dependencies unless the user asks. Keep the game as a single self-contained file.

## Code layout inside `index.html`

The game script is one IIFE with `'use strict'`. Key sections, in file order:

- **DOM refs and storage keys** — element lookups at the top; `STORAGE_*` constants prefixed `skyfighter.`.
- **Tuning tables** — `DIFFICULTIES` (easy/normal/hard: spawn rate, enemy speed, shoot chance, lives, bombs, boss HP multiplier), `KILLS_PER_STAGE`, `THEMES` (background palettes cycled per stage), `ENEMY_TYPES` (grunt, interceptor, tank), `BOSSES` (six bosses, cycled).
- **Audio** — a Web Audio synth (`ensureAudio`, `beep`, `noiseBurst`, `sfx`). No audio files.
- **Adaptive quality** — `trackPerf` lowers/raises a `quality` level (0–2) based on a rolling frame-time average.
- **Input** — keyboard (`keys` map), mouse on the canvas, and touch joystick/fire/bomb buttons shown when a touch device is detected.
- **Game state** — `resetGame`, `update(dt)`, `spawnEnemy`, `spawnBoss`, `applyPowerUp`, `useBomb`, `fireWeapon`, `loseLife`, `endGame`, `togglePause`.
- **Boss patterns** — `BOSS_PATTERNS` and the `fire*` helpers (radial, spread, spiral, cross, homing, double ring).
- **Rendering** — `draw()`, `drawPlane`, `drawBossShip`, `drawClouds`, all on a 2D context. The logical play field is `BASE_W` × `BASE_H` (480 × 720), scaled by `resize()`.
- **Loop** — `loop(now)` clamps `dt` to 50 ms, then calls `update` and `draw` while running and not paused.

Persistence is `localStorage` only (best score, mute, difficulty, reduced-motion preference). Wrap any new storage access in try/catch if you add it, since storage can be blocked.

## Conventions

- Keep the existing style: 2-space indent, single quotes in JS, `const`/`let`, small named functions, comment section headers with `// ---------- Name ----------`.
- Put gameplay numbers in the tuning tables rather than scattering magic numbers through the update loop.
- Keep accessibility hooks working: the `srLive` live region is used by `announce()`, and reduced-motion is respected via `body.reduced-motion` and the `prefers-reduced-motion` media query.
- Keep the controls hint (`#controlsHint`) in sync if you change any key bindings.
- Preserve the mobile layout: the page must work at phone width with no horizontal scroll.

## Git

- Default development branch for this task: `claude/gifted-tesla-kgvsa0`.
- Commit messages are short and imperative, describing the game change (for example, the existing commit "Add Sky Fighter: Raiden-style vertical shoot-em-up game").
- Don't open pull requests unless asked.
