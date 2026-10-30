# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a small static web project. Its only real content is **Sky Fighter**, a Raiden-style vertical shoot-'em-up, implemented as a single self-contained file:

- `index.html` (~1,670 lines): all markup, CSS (`<style>`), and game logic (one inline `<script>` wrapped in an IIFE with `'use strict'`).
- `README.md`: placeholder title only.

There is no build step, package manager, bundler, test suite, linter, or backend. Nothing needs to be installed.

## Running and verifying

- Open `index.html` directly in a browser, or serve the directory with any static server (for example `python3 -m http.server`).
- There are no automated tests or lint scripts. Verify changes by loading the game in a browser and playing through the affected path. For UI changes, check both desktop (keyboard and mouse) and touch layouts.

## Code layout inside `index.html`

- **Markup** near the top of `<body>`: `<canvas id="gameCanvas">`, the HUD (score, best, multiplier, weapon, hearts, bombs), the overlay (start, pause, and game-over screens), touch controls (`#touchControls`, `#moveZone`, `#fireBtn`, `#bombBtn`), and a visually hidden `#srLive` live region for screen readers.
- **Constants** at the top of the script:
  - `DIFFICULTIES`: spawn rate, enemy speed and fire chance, lives, bombs, and boss HP multiplier for `easy`, `normal`, and `hard`.
  - `THEMES`: background palettes. The stage number selects the theme.
  - `ENEMY_TYPES`: `grunt`, `interceptor` (weaving), and `tank` (multi-hit).
  - `BOSSES`: six stage bosses, each with colors and an HP multiplier.
  - `KILLS_PER_STAGE`: kills needed to advance a stage.
- **Audio**: `ensureAudio()` lazily creates the Web Audio context and must run from a user gesture. Sound effects are built from `beep()` and `noiseBurst()`. Sounds are collected in the `sfx` object.
- **Game state and loop**: `resetGame()` initializes state. `update(dt)` advances the simulation, `draw()` renders it, and `loop(now)` drives both through `requestAnimationFrame`. `dt` is clamped to 50 ms.
- **Weapons and power-ups**: `fireWeapon()` and `applyPowerUp()`.
- **Bosses**: `BOSS_PATTERNS` is a list of bullet-pattern functions (radial, spread, spiral, cross, homing, double-ring). `updateBoss()` cycles through them.
- **Adaptive quality**: `trackPerf()` averages the last 45 frame times and steps the `quality` level (0 full, 1 medium, 2 low) to keep the frame rate up.
- **Input**: keyboard (`keys` map), mouse on the canvas, and a touch joystick plus fire and bomb buttons.

## Persistence

The game stores four keys in `localStorage`. Do not rename them without a migration, because existing players would lose their saved settings:

| Key | Value |
| --- | --- |
| `skyfighter.bestScore` | Number |
| `skyfighter.muted` | `'1'` or `'0'` |
| `skyfighter.difficulty` | `easy`, `normal`, or `hard` |
| `skyfighter.reducedMotion` | `'1'` or `'0'`. If unset, the `prefers-reduced-motion` media query is used. |

Reads and writes to `localStorage` are currently unguarded. In browsers where storage is blocked, such as some private modes or with strict site settings, `localStorage` access can throw. If you touch this code, wrap these calls in `try`/`catch` so the game still runs.

## Conventions

- Keep everything in `index.html` unless the user asks for a split. Don't add a build step, framework, or dependency to a project this small.
- Follow the existing style: two-space indentation, `const` and `let` (no `var`), arrow functions for callbacks, and short section comments like `// ---------- Name ----------`.
- Tune game balance through the constant tables (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `THEMES`) rather than hard-coding numbers into logic.
- Keep the canvas drawn at the logical `BASE_W` × `BASE_H` (480 × 720) and scale it with `resize()`. Don't hard-code screen pixel sizes.
- New UI text that changes during play should go through `announce()` so screen readers pick it up.
- Respect `reducedMotion` for new screen effects. Shake, flash, and similar effects should check the flag.
- Keep the page working on phone-sized viewports. The layout uses `viewport-fit=cover` and touch controls.

## Git

- Develop on the branch assigned for the session (currently `claude/gifted-tesla-ms9grg`). Don't push to `main` without explicit permission.
- Write commit messages that describe the change, in the imperative mood, with a short subject line.
- Only create a pull request when the user explicitly asks for one.
