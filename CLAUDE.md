# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo contains **Sky Fighter**, a single-file, browser-based Raiden-style vertical shoot-'em-up. There is no build step, package manager, test suite, or server-side code.

Files:

- `index.html` (~1670 lines): the entire game. Inline `<style>` (lines ~10–361), the DOM/HUD/overlay markup (lines ~362–421), and one inline `<script>` IIFE (lines ~422–1668) that holds all game logic, rendering, audio, and input.
- `README.md`: a one-line title. It contains no setup instructions.

## Running and testing

- Open `index.html` directly in a browser, or serve the directory with any static server (e.g. `python3 -m http.server`).
- There are no automated tests, linters, or build commands. Verify changes by playing the game in a browser. Check at least: start, fire, bomb, pause, a stage transition, a boss fight, game over, and the mobile touch controls.

## Architecture (inside the script IIFE)

The script is wrapped in `(function () { 'use strict'; ... })()`. Keep new code inside that wrapper; do not add globals.

Major sections, roughly in source order:

1. **DOM refs**: `getElementById` lookups for every HUD, overlay, and control element. Element IDs in markup and script must stay in sync.
2. **Persistence keys**: `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion` in `localStorage`. Changing a key orphans existing player data, so change one only deliberately.
3. **Adaptive quality**: `trackPerf` samples frame time and steps `quality` (0 full, 1 medium, 2 low) to shed effects on slow devices.
4. **Tuning tables**:
   - `DIFFICULTIES` (`easy` / `normal` / `hard`): spawn rate, enemy speed, shoot chance, starting lives and bombs, boss HP multiplier.
   - `KILLS_PER_STAGE` (18): kills needed to advance a stage.
   - `THEMES`: background palettes cycled per stage.
   - `ENEMY_TYPES` (`grunt`, `interceptor`, `tank`) and `BOSSES` (six bosses, each with a `hpMul`).
   - `BOSS_PATTERNS`: bullet-pattern functions for bosses (radial, aimed spread, spiral, cross, homing, double ring, etc.).
5. **Audio**: a Web Audio API synth. `ensureAudio()` must run from a user gesture before any sound plays. Helpers are `beep` and `noiseBurst`; `sfx` holds named effects.
6. **Input**: keyboard (`keys` map keyed by `e.code`), mouse on the canvas, and touch joystick and buttons (`moveZone`, `fireBtn`, `bombBtn`).
7. **Game state**: `resetGame()` initializes `player`, enemies, bullets, and power-ups. `update(dt)` advances the simulation. `draw()` renders. `loop(now)` drives both via `requestAnimationFrame`, with `dt` clamped to 0.05 s.
8. **Gameplay functions**: `fireWeapon`, `applyPowerUp`, `useBomb`, `breakChain`/`addScore` (combo multiplier), `updateBoss`/`killBoss`, `loseLife`, `endGame`, `togglePause`.

Canvas is a fixed 480×720 logical resolution (`BASE_W`, `BASE_H`). `resize()` scales the CSS size to fit the window, capped at 1.6×. Write all game coordinates in logical pixels.

## Conventions

- Plain ES2015+ JavaScript and CSS, no frameworks, no dependencies, no bundler. Do not introduce any.
- Keep everything in `index.html` unless the user asks for a split. Keep the single-file, zero-dependency property.
- Use `const` by default and `let` where reassigned. Match the existing 2-space indentation and single-quoted strings.
- Add new tuning values to the existing tables (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, etc.) rather than hard-coding magic numbers in logic.
- Respect accessibility: maintain `aria-live` announcements via `announce()`, and gate screen shake, flashes, and pulses behind `reducedMotion`.
- Keep touch and keyboard controls working together. Any new action needs both a key binding and a touch control, or a documented reason it has none.
- `controlsHint` text in the overlay documents the current controls. Update it when bindings change.

## Controls (current)

- Move: mouse (steer on canvas), arrow keys or WASD, touch joystick (drag left).
- Fire: Space (hold), or hold the mouse, or FIRE button.
- Bomb: X or Shift, or BOMB button.
- Pause: P or Esc, or the pause button.
- Mute and reduced-motion toggles are in the top bar.

## Working in this repo

- Work on the branch the session specifies. Do not push to other branches without permission.
- Make small, focused commits with clear messages describing gameplay or code changes.
- Before finishing a gameplay change, play it through in a browser. Syntax errors in the single inline script will blank the whole game, so reload and check the console.
- Do not commit generated files, screenshots, or local tooling output.
