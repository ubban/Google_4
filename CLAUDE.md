# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo is a single-page browser game, **Sky Fighter**: a Raiden-style vertical shoot-em-up.

- `index.html` is the entire game: markup, CSS (`<style>`, roughly lines 10–359), and JavaScript (`<script>`, roughly lines 422–1667). There is no build step, no bundler, no package manager, and no test suite.
- `README.md` is a placeholder with no setup instructions.
- Git: develop on the branch assigned to the task (currently `claude/gifted-tesla-k43oii`). Do not push to other branches without explicit permission.

## Running the game

No build is required. Open `index.html` directly in a browser, or serve the directory with any static server (for example `python3 -m http.server`) and visit it.

Verify changes by playing the game in a browser. There are no automated tests or linters configured, so state in your summary what you actually exercised.

## Code layout inside `index.html`

The script is one IIFE with `'use strict'`. Nothing is exported and nothing is global, so every function and constant lives inside that closure. Keep new code inside it.

Major sections, in order of appearance:

| Section | What it holds |
|---|---|
| DOM refs (top of IIFE) | `document.getElementById` lookups for every HUD/overlay element. |
| `STORAGE_*` constants | `localStorage` keys: `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion`. |
| `announce`, `trackPerf` | Screen-reader live-region helper; adaptive quality (`quality` 0–2) driven by a 45-frame average of frame time. |
| `DIFFICULTIES` | Per-difficulty tuning: spawn rate, enemy speed, shoot chance, lives, bombs, boss HP multiplier. |
| `KILLS_PER_STAGE` (18) | Kills needed before a boss warning and fight. |
| `THEMES` | Six background palettes, cycled by `(stage - 1) % THEMES.length`. |
| `ENEMY_TYPES` | `grunt`, `interceptor` (weaving), `tank` (4 HP, unlocks at stage 2). |
| `BOSSES` | Six boss definitions (VANGUARD → TITAN) with colors and HP multipliers. |
| Audio (`ensureAudio`, `beep`, `noiseBurst`, `sfx`) | Web Audio synthesis. No audio files. Mute is persisted. |
| `resize` | Fits a fixed 480×720 logical canvas to the window, capped at 1.6× scale. |
| Touch / mouse input | Virtual joystick and fire/bomb buttons for touch; mouse aim and fire for desktop. |
| `resetGame`, `spawnEnemy`, `spawnBoss`, `fireWeapon`, `useBomb` | Game-state setup and actions. |
| `update(dt)` | Per-frame simulation: movement, collisions, scoring, chain/combo, stage progression. |
| `applyPowerUp` | `life`, `bomb`, and `weapon` pickups. Weapons are `vulcan` and `laser`, each with levels 1–3. |
| `BOSS_PATTERNS`, `updateBoss`, `killBoss` | Boss bullet patterns (radial, aimed spread, spiral, cross, homing, double ring) and boss lifecycle. |
| `loseLife`, `endGame`, `togglePause` | Game-over and pause flow. |
| `draw()` | All canvas rendering. Honors `quality` and reduced-motion. |
| `loop(now)` | `requestAnimationFrame` loop. `dt` is clamped to 0.05 s. |

## Controls

- **Keyboard:** Arrow keys or WASD to move. Space to fire (hold). `X` or Shift to bomb. `P` or Escape to pause.
- **Mouse:** The ship follows the cursor over the canvas. Pressing any button on the canvas starts firing, and releasing it anywhere stops firing.
- **Touch:** Virtual joystick on the left, fire button and bomb button on the right.

## Conventions

- **Single file.** Keep HTML, CSS, and JS in `index.html` unless the user asks for a split. Do not add a build step or dependencies without asking.
- **Match the existing style.** Two-space indentation, single quotes in JS, `const` by default, arrow functions for callbacks, and `// ---------- Section ----------` banners between major sections.
- **Keep the IIFE closed.** Do not add top-level globals.
- **Persist settings through the `STORAGE_*` keys** and wrap new `localStorage` access so the game still works when storage is unavailable.
- **Respect reduced motion.** Effects such as screen shake and flashes must check the `reducedMotion` flag and the `reduced-motion` body class.
- **Accessibility.** Announce important state changes through `announce()` so the `srLive` region picks them up.
- **Balance changes** belong in the data tables (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `BOSS_PATTERNS`) rather than being hard-coded in logic.
- **Logical coordinates.** Game logic uses the 480×720 base space. Do not use CSS pixels in simulation code.

## Git

- Write clear, descriptive commit messages in the imperative mood.
- Push with `git push -u origin <branch-name>`.
- Do not open a pull request unless the user asks for one.
