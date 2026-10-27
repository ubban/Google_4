# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Google_4 is a single-page HTML5 canvas game, **Sky Fighter**: a Raiden-style vertical shoot-em-up. The entire game lives in `index.html` (about 1,670 lines: CSS in `<style>`, markup in `<body>`, and one IIFE in a `<script>` block). There is no build step, no bundler, no package manager, and no test suite. `README.md` is a stub.

Open `index.html` directly in a browser to run the game.

## Repository layout

- `index.html`: the whole game (styles, HUD/overlay markup, and game logic).
- `README.md`: placeholder.
- `CLAUDE.md`: this file.

## Code structure inside `index.html`

The game script is wrapped in `(function () { 'use strict'; ... })();`. Everything is private to that closure, so nothing is exposed on `window`.

Key sections, in source order:

- **DOM references**: `const` lookups by id at the top of the IIFE.
- **Persistence keys**: `STORAGE_BEST`, `STORAGE_MUTE`, `STORAGE_DIFF`, and `STORAGE_REDUCED`, all prefixed `skyfighter.`. Reads go through `localStorage`.
- **Adaptive quality**: `trackPerf` samples frame times and steps a `quality` level (0 full, 1 medium, 2 low) to hold roughly 40 to 56 fps.
- **Config tables**:
  - `DIFFICULTIES` (`easy` / `normal` / `hard`): spawn rate, enemy speed and fire chance, lives, bombs, and boss HP multiplier.
  - `KILLS_PER_STAGE` (18): kills needed to advance a stage.
  - `THEMES` (6 sky palettes, cycled per stage).
  - `ENEMY_TYPES` (`grunt`, `interceptor`, `tank`), each with `minStage` gating.
  - `BOSSES` (6, each with its own palette and `hpMul`).
  - `BOSS_PATTERNS` (bullet patterns used by bosses).
- **Audio**: `ensureAudio`, `beep`, and `noiseBurst`, built on the Web Audio API. Audio starts only after a user gesture.
- **Input**: keyboard (`keys` map keyed by `e.code`), mouse (canvas steering and click-to-fire), and touch (joystick on the left, FIRE and BOMB buttons on the right).
- **Game state and logic**: `resetGame`, `update(dt)`, `fireWeapon`, `useBomb`, `applyPowerUp`, `updateBoss`, `killBoss`, `loseLife`, `endGame`, and `togglePause`.
- **Rendering**: `draw()`, plus `drawPlane`, `drawBossShip`, and `drawClouds`.
- **Loop**: `loop(now)` clamps `dt` to 50 ms and runs `update` and `draw` only while `running && !paused`.

## Controls (for reference when changing input code)

- Keyboard: arrows or WASD to move, Space to fire, X or Shift for a bomb, P or Esc to pause.
- Mouse: move to steer, click to fire.
- Touch: drag left to fly, tap FIRE, tap BOMB.

## Conventions

- **Keep it one file.** New game code goes in the existing IIFE in `index.html`. Do not add a build step, framework, or external dependency unless the user asks for it.
- **Match the local style.** Use `const` and `let`, arrow functions for callbacks, two-space indentation, single quotes in JS, and the existing section banner comments (`// ---------- Name ----------`).
- **Config over magic numbers.** Tunable values belong in the config tables (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, and so on), not inline.
- **Respect reduced motion.** Screen shake, flash, and pulse effects should check `reducedMotion` (or the `reduced-motion` body class) so the toggle keeps working.
- **Keep the DOM and canvas in sync.** HUD values are written through the cached element references. Do not query the DOM inside the per-frame loop.
- **Accessibility.** Announcements for screen readers go through `announce()`, which writes to the `#srLive` region.
- **Persistence is best-effort.** Keep `localStorage` reads and writes tolerant of missing or invalid values, as the existing `Number(...) || 0` pattern does.

## Verifying changes

There is no automated test or lint setup. To verify a change:

1. Open `index.html` in a browser (or serve the directory with any static server).
2. Play through at least one stage and check the affected behavior: movement, firing, bombs, power-ups, bosses, pause, game over, and the best-score persistence on reload.
3. Check the touch layout at phone width if the change touches HUD, overlay, or controls.
4. Check the browser console for errors.

For a syntax-only check of the inline script, extract the `<script>` block and run `node --check` on it.

## Git

- Development branch for this session: `claude/gifted-tesla-bmhrf7`.
- Commit with clear, descriptive messages. Push with `git push -u origin <branch>`.
- Do not open a pull request unless the user asks for one.
