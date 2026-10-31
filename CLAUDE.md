# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo contains **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs entirely in the browser as a single static HTML file.

- `index.html` — the whole game: inline CSS (`<style>`), markup (HUD, touch controls, start/pause overlay), and one inline `<script>` (an IIFE with `'use strict'`) containing the game loop, rendering, audio, and input.
- `README.md` — minimal placeholder; there is no other documentation.

There is no build step, no package manager, no bundler, no framework, and no test suite. Do not add one unless asked.

## Running the game

Open `index.html` directly in a browser, or serve the directory with any static server (e.g. `python3 -m http.server` and visit `http://localhost:8000`). No install is required.

## Architecture (inside `index.html`)

The script is organized by `// ----------` section comments and `const`/`function` blocks. Key pieces:

- **DOM references** — all elements are fetched once at the top of the IIFE by id.
- **Persistence** — `localStorage` keys, all prefixed `skyfighter.`:
  - `skyfighter.bestScore`
  - `skyfighter.muted`
  - `skyfighter.difficulty`
  - `skyfighter.reducedMotion`
  
  Every read/write should tolerate missing or blocked storage gracefully.
- **Adaptive performance** — `trackPerf(dt)` samples the last 45 frame times and steps a `quality` level (0 full, 1 medium, 2 low) up or down based on average frame time.
- **Tuning data** — declarative constants near the top of the script:
  - `DIFFICULTIES` (easy / normal / hard): spawn rates, enemy speed, shoot chance, lives, bombs, boss HP multiplier.
  - `KILLS_PER_STAGE` (18): kills needed before a stage's boss is triggered. The stage counter advances when the boss is defeated (`killBoss()`).
  - `THEMES`: six background palettes, indexed by `(stage - 1) % 6`.
  - `ENEMY_TYPES`: `grunt`, `interceptor` (weaving), `tank` (tougher, from stage 2).
  - `BOSSES`: six boss designs (VANGUARD → TITAN), with HP multipliers and colors.
  - `BOSS_PATTERNS`: bullet-pattern functions (radial, aimed spread, columns, spiral, cross, homing, double ring).
- **Audio** — a lazily created `AudioContext` (`ensureAudio()`), with `beep()` and `noiseBurst()` primitives and an `sfx` object of named effects. Audio starts only after a user gesture.
- **Input** — keyboard (arrows/WASD move, Space fire, X/Shift bomb, P/Esc pause), mouse (move to steer, click to fire), and touch (left-side joystick, FIRE and BOMB buttons). Touch and mouse paths are separate listeners.
- **Game state** — the `player`, `bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`, and `boss` variables are reset in `resetGame()`.
- **Main loop** — `loop(now)` clamps `dt` to 50 ms, calls `update(dt)` then `draw()` when running and not paused, and re-queues itself via `requestAnimationFrame`. `update` handles movement, firing, collisions, spawning, and stage progression; `draw` renders everything to a fixed-size 480×720 logical canvas (`BASE_W`, `BASE_H`) scaled by `resize()`.
- **Power-ups** — dropped by destroyed enemies (`maybeSpawnPowerUp`): `life`, `bomb`, and `weapon`. Weapon power-ups raise the level (1–3) of the current weapon, then switch between `vulcan` and `laser`.
- **Accessibility** — an `aria-live` region (`#srLive`, written via `announce()`), a reduced-motion toggle that adds `body.reduced-motion` (disables screen shake, flashes, and pulses), and `prefers-reduced-motion` as the default.

## Conventions

- Keep the game in a single file unless the user asks for a restructure. Keep CSS in `<style>` and JS in the existing IIFE.
- Match the existing style: 2-space indentation, single quotes in JS, `const` by default, arrow functions for callbacks, and section comments in the `// ---------- Name ----------` form.
- Put new tuning values in the existing data constants (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, etc.) rather than hard-coding them in logic.
- New `localStorage` keys must use the `skyfighter.` prefix and be declared alongside the existing `STORAGE_*` constants.
- Any new visual effect should respect `reducedMotion` and, where it is expensive, the `quality` level.
- Any new interactive control needs keyboard, mouse, and touch paths where applicable, plus an accessible label or `title`.

## Verification

There are no automated tests or linters configured. To verify a change:

1. Open `index.html` in a browser and check the browser console for errors.
2. Play through start → stage clear → boss → game over for the affected area.
3. Check all three difficulty levels if gameplay numbers changed.
4. Check touch controls with browser device emulation if input code changed.
5. Confirm pause (P/Esc), mute, and reduced-motion toggles still work and persist across reloads.

## Git workflow

- Default branch: `main`.
- Commit messages: short imperative subject line describing the change.
- Do not commit generated or editor files; the repo currently contains only `README.md`, `index.html`, and this file.
