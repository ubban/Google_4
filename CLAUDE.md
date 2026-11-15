# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Google_4 is a small static web project containing **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs in the browser on a single HTML `<canvas>`.

- `index.html` — the entire game: markup, CSS (`<style>`, lines ~10–360), and JavaScript (`<script>`, lines ~422–1668), all inline.
- `README.md` — a placeholder title only.

There is no build step, no bundler, no `package.json`, no test suite, and no linter configured. Nothing needs to be installed.

## Running the game

Open `index.html` directly in a browser, or serve the directory with any static server (for example `python3 -m http.server` from the repo root) and visit it. Nothing else is required.

## Code structure (`index.html`)

The script is one IIFE (`(function () { 'use strict'; ... })();`) with no globals. Major sections, in source order:

- **DOM references and constants** — element lookups, `STORAGE_*` localStorage keys (`skyfighter.*`), `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES` (background palettes per stage), `ENEMY_TYPES` (grunt, interceptor, tank), `BOSSES` (six boss definitions).
- **Persistence** — best score, mute, difficulty, and reduced-motion preference are read from and written to `localStorage`.
- **Adaptive performance** — `trackPerf` samples frame times and steps a `quality` level (0–2) up or down to keep frame rate up; drawing code consults it.
- **Audio** — a Web Audio synth (`ensureAudio`, `beep`, `noiseBurst`, and an `sfx` table). Audio is created lazily on the first user gesture.
- **HUD and feedback** — hearts, bombs, combo pop, stage banner, screen shake, flash/vignette, and an ARIA live region (`announce`) for screen readers.
- **Input** — keyboard (arrows/WASD to move, Space or the fire control to shoot, X/Shift for bombs, P/Escape to pause), mouse on the canvas, and on-screen touch controls (joystick, fire, bomb).
- **Game state** — `resetGame()` builds the `player` and related arrays; `update(dt)` advances the simulation; `draw()` renders everything.
- **Weapons and power-ups** — `vulcan` and `laser` weapons (`fireWeapon`), weapon-swap power-ups (`applyPowerUp`), and the bomb (`useBomb`).
- **Enemies and bosses** — `spawnEnemy`, `spawnBoss`, the radial/spread/homing/spiral attack patterns in `BOSS_PATTERNS`, `updateBoss`, and `killBoss`.
- **Flow** — `loseLife`, `endGame`, `togglePause`, and the main `loop(now)` driven by `requestAnimationFrame`. `loop` clamps `dt` to 50 ms.

## Conventions

- Keep the game self-contained in `index.html` unless there is a clear reason to split it. If it is split, update this file's structure section.
- Tunable numbers (difficulty, enemy stats, boss HP multipliers, spawn rates) live in the constant tables near the top of the script. Prefer changing those over hard-coding values deeper in the logic.
- Preserve the `'use strict'` IIFE scoping; do not introduce new globals.
- Keep the code as plain, dependency-free JavaScript and CSS. Do not add external libraries without a specific request.
- Respect the `prefers-reduced-motion` preference and the existing mute/reduced-motion toggles when adding visual or audio effects.
- Keep the layout usable on touch devices; the touch controls are part of the game, not an afterthought.

## Verification

Because there is no test harness, verify changes by loading `index.html` in a browser and playing through the affected behavior (start, shoot, bomb, pause, a stage transition, a boss, game over). For visual or input changes, check both keyboard and touch layouts. If a headless browser is used, it should be checked against the actual page rather than assumed to work.

## Git

- Default branch work should go on a feature branch; do not commit directly to the default branch.
- Write clear, present-tense commit messages that describe the gameplay or code change.
