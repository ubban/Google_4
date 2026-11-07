# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo is a small, dependency-free web game: **Sky Fighter**, a Raiden-style vertical shoot-'em-up.

- The entire game lives in a single file, `index.html` (~1,670 lines): inline `<style>`, markup, and one `<script>` IIFE that runs on an HTML5 `<canvas>` (2D context).
- `README.md` is a stub. There is no build step, no package manager, no test suite, and no linter configured.

## Running and testing

- Open `index.html` directly in a browser, or serve the directory with any static server (e.g. `python3 -m http.server`).
- There are no automated tests. Verify changes by playing the game in a browser, ideally at both desktop and phone-sized viewports (the layout and touch controls are responsive).

## Code structure (`index.html`)

- **Markup/CSS (lines ~1–420):** HUD (`#hud` with score, best, weapon, multiplier, hearts, bombs), overlay card (`#overlay`), touch controls (`#touchControls`, `#joystickBase`, `#fireBtn`, `#bombBtn`), boss bar (`#bossBarWrap`), vignette/flash effects, and a screen-reader live region (`#srLive`).
- **Script (lines ~422–end):** everything sits inside one IIFE. Key sections, roughly in file order:
  - DOM references (`const xxxEl = document.getElementById(...)`).
  - Tunables: `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES` (per-stage sky palettes), `ENEMY_TYPES`, `BOSSES`.
  - Persistence via `localStorage` (see below).
  - Audio: lazily created `AudioContext`, with `beep()` and `noiseBurst()` helpers. Sound is synthesized, with no audio files.
  - Input: keyboard (`keys` map, `keydown`/`keyup`), mouse, and touch (joystick + buttons).
  - Rendering: `drawPlane`, `drawBossShip`, cloud/parallax layers, explosions, and HUD updates.
  - Simulation: `update(dt)`, `updateBoss(dt)`, `spawnEnemy()`, `spawnBoss()`, `spawnExplosion()`.
  - Rendering entry: `draw()`. Main loop: `loop(now)` driven by `requestAnimationFrame`.
  - `trackPerf()` adaptively drops render `quality` (0 full / 1 medium / 2 low) when frame times slip.

## Gameplay model (for reference when changing balance)

- **Stages:** each stage is `KILLS_PER_STAGE` (18) kills and then a boss. Themes and bosses cycle by `(stage - 1) % length`.
- **Difficulty** (`easy` / `normal` / `hard`) controls spawn rates, enemy speed, enemy fire chance, starting lives and bombs, and boss HP multiplier.
- **Weapons:** the `weapon` power-up toggles between Vulcan and Laser, and the `level` field tracks weapon power.
- **Bombs:** limited smart bombs that clear the screen and damage bosses.
- **Scoring:** a kill-chain combo multiplier.

## Controls

- Mouse: move to steer, click to fire.
- Arrows / WASD: move. Space: fire. X or Shift: bomb. P or Esc: pause.
- Touch: drag on the left to fly, tap FIRE, tap BOMB.

## Persistence keys

All stored in `localStorage` under the `skyfighter.` prefix:

| Key | Meaning |
|---|---|
| `skyfighter.bestScore` | High score |
| `skyfighter.muted` | `'1'` when audio is muted |
| `skyfighter.difficulty` | Selected difficulty |
| `skyfighter.reducedMotion` | Explicit reduced-motion override (absent means follow the OS preference) |

Changing these keys breaks saved player data, so treat them as a stable interface.

## Conventions

- Keep the game as a single self-contained HTML file unless the user asks for a restructure. Do not add dependencies, a bundler, or a build step without asking.
- Match the existing style: 2-space indentation, `const`/`let`, DOM refs at the top of the IIFE, and small named helper functions.
- Respect `reducedMotion` for any new screen shake, flashing, or large animation.
- Keep touch and keyboard controls in parity when adding input.
- `localStorage` reads and writes are currently not wrapped in `try/catch`. If you add new storage access, consider wrapping it, since storage can throw in private or blocked-storage contexts.

## Git

- Development branch for this work: `claude/gifted-tesla-3bcv92`.
- Commit messages are descriptive and start with a summary line (see `git log`).
