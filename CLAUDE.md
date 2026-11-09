# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo is **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs in the browser on a `<canvas>`. The whole game is one self-contained file:

- `index.html` (~1,670 lines) — all markup, CSS (`<style>` at ~line 10), and game logic (`<script>` at ~line 422, wrapped in an IIFE with `'use strict'`).
- `README.md` — minimal placeholder; contains no setup or usage instructions.

There is no package manager, build step, bundler, test suite, linter, or CI configuration. There is no `package.json`.

## Running and testing

- **Run:** open `index.html` directly in a browser, or serve the directory with any static server (for example `python3 -m http.server` from the repo root and visit `http://localhost:8000`).
- **Tests:** none exist. Verify changes by playing the game in a browser. For automated checks, a Playwright script that loads the page and exercises the start button and keyboard/touch input is a reasonable approach.
- **Lint/format:** none configured. Match the existing style (2-space indentation, single quotes in JS, `const`/`let`, semicolons).

## Code layout (inside `index.html`)

The script is organized into sections marked by `// ---------- Name ----------` comments and grouped functions. Key pieces:

| Area | Where / what |
| --- | --- |
| DOM references | Top of the script: `canvas`, HUD elements, buttons, overlay, touch controls. IDs are referenced via `getElementById`, so renaming an element ID means updating the script too. |
| Storage keys | `STORAGE_BEST`, `STORAGE_MUTE`, `STORAGE_DIFF`, `STORAGE_REDUCED` (all prefixed `skyfighter.`). |
| Adaptive quality | `trackPerf()` samples frame times and steps `quality` (0 full, 1 medium, 2 low). |
| Config tables | `DIFFICULTIES` (spawn rates, enemy speed, shoot chance, lives, bombs, boss HP multiplier), `KILLS_PER_STAGE` (18), `THEMES` (per-stage sky palettes), `ENEMY_TYPES` (grunt, interceptor, tank), `BOSSES` (six named bosses). |
| Audio | `ensureAudio()`, `beep()`, `noiseBurst()`, and the `sfx` object. Audio is created lazily on the first user gesture, which browsers require. |
| Input | Keyboard listeners on `window`; a virtual joystick and FIRE/BOMB buttons for touch; mouse/pointer handling on the canvas. |
| Game state | `resetGame()` initializes state. `update(dt)` advances the simulation; `draw()` renders. `loop(now)` drives both via `requestAnimationFrame`, with `dt` clamped to 0.05 s. |
| Combat | `fireWeapon()`, `applyPowerUp()`, `useBomb()`, `spawnEnemy()`, `spawnBoss()`, `updateBoss()`, `killBoss()`, and `BOSS_PATTERNS` (bullet patterns such as radial, spread, spiral, cross, homing, double ring). |
| Scoring | `addScore()` applies the score multiplier; `breakChain()` resets the chain. |
| Lifecycle | `loseLife()`, `endGame()` (writes best score to storage), `togglePause()`. |
| Rendering | `drawPlane()`, `drawBossShip()`, `drawClouds()`, and `draw()`. Drawing uses canvas 2D primitives only. |

Game logical size is `BASE_W = 480` by `BASE_H = 720`. `resize()` scales the canvas to fit the viewport.

## Conventions and constraints

- **Keep it a single file** unless the user asks for a restructure. The project is deliberately dependency-free and opens straight from disk.
- **No external dependencies.** Do not add CDN scripts, fonts, or npm packages without asking. The page currently loads nothing external; the favicon is an inline SVG data URI.
- **Persist state only through the existing `skyfighter.*` localStorage keys.** Wrap new storage reads and writes defensively, since storage can be unavailable in private windows or when site data is blocked.
- **Accessibility:** keep the `srLive` live region updated through `announce()` for important game events. Respect the reduced-motion toggle (`reducedMotion`) and the `prefers-reduced-motion` media query when adding screen shake, flashes, or pulsing effects.
- **Touch and desktop parity:** any new control needs both a keyboard/mouse path and a touch path, and the touch controls must not block scrolling or zoom outside the game area.
- **Pause and game-over states:** new per-frame logic belongs in `update()`, and new drawing belongs in `draw()`. Anything that should freeze must respect the `running && !paused` guard in `loop()`.
- **Balance changes** should go in the config tables (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `KILLS_PER_STAGE`) rather than being hard-coded in logic.

## Git workflow

- Default branch is `main`. Feature work for this environment is developed on `claude/gifted-tesla-eh0nsa`.
- Commit with clear, descriptive messages. Push with `git push -u origin <branch>`.
- Do not open pull requests unless the user explicitly asks for one.
