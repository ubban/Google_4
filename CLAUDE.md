# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Overview

This repo is a small static web project. Its only real content is **Sky Fighter**, a
Raiden-style vertical shoot-'em-up written as a single-file HTML5 canvas app.

- `index.html` (~1,670 lines): the entire game. Markup, CSS, and JavaScript are all inline.
- `README.md`: a placeholder title. It has no setup or usage instructions.

There is no build step, package manager, test suite, linter, or CI configuration.

## Running the game

Open `index.html` directly in a browser, or serve the directory with any static file
server. No dependencies are loaded from the network.

## File layout of `index.html`

1. `<head>`: meta tags (viewport locked for mobile, theme color), an inline SVG favicon,
   and a `<style>` block with all CSS.
2. `<body>`: a `#container` holding the `#gameCanvas`, the HUD (`#hud`, `#topBar`,
   `#bossBarWrap`), transient effects (`#comboPop`, `#stageBanner`, `#flash`,
   `#vignette`), touch controls (`#touchControls`), and the `#overlay` start/pause/game-over card.
3. A single `<script>` containing one IIFE (`'use strict'`). All game state and logic
   live inside it. Nothing is exported to `window`.

## Architecture of the game script

The script is organized by section comments and roughly follows this order:

- **DOM references and storage keys.** Elements are looked up once at the top.
  Persistent settings use `localStorage` under the `skyfighter.*` prefix:
  `bestScore`, `muted`, `difficulty`, `reducedMotion`.
- **Adaptive performance.** `trackPerf` averages the last 45 frame times and steps a
  `quality` level (0 full, 1 medium, 2 low) up or down.
- **Tuning tables.**
  - `DIFFICULTIES`: spawn rate, enemy speed, enemy fire chance, starting lives and
    bombs, and boss HP multiplier for `easy`, `normal`, and `hard`.
  - `KILLS_PER_STAGE` (18): kills needed to advance a stage.
  - `THEMES`: six sky palettes that cycle with stage.
  - `ENEMY_TYPES`: `grunt`, `interceptor` (weaves), and `tank` (4 HP, appears from stage 2).
  - `BOSSES`: six boss designs with colors and an HP multiplier.
- **Audio.** Synthesized with the Web Audio API (`beep`, `noiseBurst`, and an `sfx` object).
  Nothing is loaded from disk. `ensureAudio()` must run from a user gesture.
- **Input.** Keyboard (`keys` map keyed by `e.code`), mouse on the canvas, and a
  touch joystick plus fire and bomb buttons.
- **Game state and loop.** `resetGame()` initializes state. `update(dt)` advances
  the simulation, and `draw()` renders it. `loop(now)` is the `requestAnimationFrame`
  driver. `dt` is clamped to 50 ms, and nothing updates while paused or not running.
- **Weapons.** `fireWeapon()` supports two weapons, `vulcan` (spread shots) and
  `laser` (piercing beams), each with 3 levels. `applyPowerUp()` handles the power-up
  types `life`, `bomb`, and `weapon`.
- **Stages and bosses.** Reaching `KILLS_PER_STAGE` kills schedules a boss about 1.2 s
  later. The boss design is `BOSSES[(stage - 1) % BOSSES.length]`, and the theme is
  `THEMES[(stage - 1) % THEMES.length]`. `BOSS_PATTERNS` is a list of bullet-pattern
  functions (radial, aimed spread, columns, spiral, cross, homing, double ring) that
  `updateBoss()` cycles through. `killBoss()` advances `stage`, and `endGame()` ends the run.
- **Quality levels.** `quality` also scales particle counts and shadow/glow effects in
  `draw()`, so visual changes should respect it.

## Controls

| Action        | Keyboard                          | Touch / mouse                  |
|---------------|-----------------------------------|--------------------------------|
| Move          | Arrow keys or WASD                | Left joystick / mouse on canvas |
| Fire          | Space (hold)                      | Fire button (hold)             |
| Smart bomb    | X or Shift                        | Bomb button                    |
| Pause         | P or Escape                       | Pause button                   |

Space is `preventDefault`-ed so it does not scroll the page.

## Conventions

- Keep everything in `index.html` unless there is a clear reason to split it. The
  single-file design is intentional for this project.
- Match the existing style: two-space indentation, single quotes in JS, `const` by
  default, and short section comments using `// ---------- Name ----------`.
- Put new tuning values in the existing tables (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`,
  `THEMES`) rather than hard-coding them in logic.
- Use `localStorage` only through the `skyfighter.*` keys. Keep reads and writes
  guarded, because private or restricted browsing can throw.
- Respect `reducedMotion`. Screen shake, flashes, and similar effects should check it.
- Keep UI text that screen readers need flowing through `announce()` and the `#srLive` region.
- Support both mouse/keyboard and touch. New controls need touch equivalents.

## Verifying changes

There are no automated tests. After editing, open `index.html` in a browser and check:

1. The start overlay appears, and a game starts and ends with a score.
2. Keyboard, mouse, and touch controls all work. Check touch with a narrow viewport.
3. Pause and resume work, including from the overlay button.
4. A stage clears, a boss appears, and the boss health bar updates.
5. The best score and settings persist after a reload.

Check that the JavaScript has no syntax errors, for example in the browser console or
with `node --check` on the extracted script.

## Git

- Development happens on feature branches named `claude/...`. Do not push to other
  branches without permission.
- Use clear, descriptive commit messages that explain the gameplay or UI change.
