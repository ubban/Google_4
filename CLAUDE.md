# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo is a single-page browser game, **Sky Fighter**: a Raiden-style vertical shoot-em-up. The whole game lives in one file, `index.html`, with inline CSS (`<style>`, ~lines 10–361), markup (~lines 362–421), and one inline JavaScript block (`<script>`, ~lines 422–1669).

Files:

- `index.html`: the complete game (markup, styles, and logic).
- `README.md`: a placeholder, currently just the project title.

There is no build step, package manager, bundler, test suite, or linter. Do not add one unless asked.

## Running and verifying

- **Run:** open `index.html` directly in a browser. No server is required.
- **Syntax check** (fast, no browser needed). Extract the inline script and run it through Node:
  ```bash
  awk '/<script>/{f=1;next}/<\/script>/{f=0}f' index.html > /tmp/game.js && node --check /tmp/game.js
  ```
  Use the session scratchpad instead of `/tmp` when working in a Claude Code session.
- For gameplay or visual changes, actually play the game in a browser (desktop and a narrow/touch viewport) rather than relying on the syntax check alone.

## Architecture

Everything runs inside one IIFE (`(function () { 'use strict'; ... })();`). Keep new code inside it so names don't leak to `window`.

Key sections, in source order:

- **DOM references** (`const canvas = …`, etc.): all element lookups happen once, at the top.
- **Persistence keys** (`STORAGE_*`): `localStorage` keys prefixed `skyfighter.`. Stored values: best score, mute, difficulty, reduced motion.
- **Adaptive quality** (`trackPerf`): samples frame times and lowers/raises a `quality` level (0 full, 1 medium, 2 low) to keep the frame rate up.
- **Tuning tables**:
  - `DIFFICULTIES`: spawn rates, enemy speed and fire chance, lives, bombs, and boss HP multiplier for `easy`, `normal`, `hard`.
  - `KILLS_PER_STAGE` (18): kills needed before a boss spawns.
  - `THEMES`: background palettes, cycled per stage.
  - `ENEMY_TYPES`: `grunt`, `interceptor` (weaving), `tank` (4 HP). Each has a `minStage` gate.
  - `BOSSES`: six bosses, cycled by stage. Each has its own palette and `hpMul`.
- **Audio** (`ensureAudio`, `beep`, `noiseBurst`, `sfx.*`): synthesized with the Web Audio API. No audio files. `ensureAudio()` must run from a user gesture before sound can play.
- **Input**: keyboard (`keys[e.code]`), mouse on the canvas, and touch (virtual joystick, fire, and bomb buttons).
- **Game state**: module-level `let` variables (`player`, `enemies`, `bullets`, `enemyBullets`, `powerUps`, `particles`, `boss`, `score`, `lives`, `stage`, …). `resetGame()` is the single place that initializes them. Add any new state there too.
- **Update / draw loop**: `update(dt)` advances the simulation, `draw()` renders, and `loop(now)` is the `requestAnimationFrame` driver. `update` runs with a clamped `dt`.
- **Bosses**: `BOSS_PATTERNS` is an array of attack functions (`fireRadial`, `fireSpiralBurst`, `fireHomingSeeded`, …). `updateBoss` picks patterns, and `killBoss` advances the stage.

## Conventions

- **Language and style:** plain ES2017+ JavaScript, no modules, no TypeScript, no framework. Match the existing style: 2-space indent, single quotes, `const`/`let` (no `var`), arrow functions for callbacks, and `function` declarations for named helpers.
- **Comments:** section headers use the `// ---------- Name ----------` form. Keep them brief, and only comment non-obvious logic.
- **Tuning values** live in the constant tables above, not as magic numbers inside update logic. Add new enemy, boss, or difficulty parameters to those tables.
- **Rendering:** draw with the 2D canvas context. Coordinates are in the `BASE_W` × `BASE_H` (480 × 720) logical space; `resize()` scales the canvas to fit the window.
- **Accessibility:** respect `reducedMotion` (disables or softens screen shake, flashes, and similar effects). Announce important state changes through `announce()`, which writes to the `aria-live` region.
- **Responsive/touch:** every new control must work on touch as well as keyboard and mouse. Keep the layout usable at phone width.
- **No external dependencies:** do not add CDN scripts, fonts, or npm packages without asking. The one `<link rel="icon">` is an inline data URI; keep it that way.

## Controls (for reference when testing)

| Action | Keyboard | Touch |
| --- | --- | --- |
| Move | Arrow keys or WASD | Left-side joystick |
| Fire | Space (hold) | Fire button |
| Bomb | X or Shift | Bomb button |
| Pause | P or Escape | Pause button |

## Gameplay summary

- Three lives by default on Normal (4 on Easy); the player starts with a bomb stock set by difficulty.
- Enemies spawn on a timer that tightens over elapsed play time (every 300 frames) down to a per-difficulty minimum (`spawnMin`). Kills build a chain for score multipliers; the chain breaks if no kill lands within about 2.2 seconds (`chainTimer`).
- Power-ups: `life`, `bomb`, and `weapon`. Weapon upgrades go level 1 → 3, then switch between `vulcan` and `laser`.
- Every 18 kills (`KILLS_PER_STAGE`) a boss appears. Beating it advances the stage, switches the background theme, and cycles to the next boss.
- Best score is stored in `localStorage` under `skyfighter.bestScore`.

## Working agreements

- Keep changes small and in the style of the surrounding code. Don't refactor the whole file while fixing one thing.
- When changing gameplay numbers, update the constant tables rather than hard-coding values in logic.
- After any change to `index.html`, run the syntax check above, then play the affected path in a browser before reporting it as working.
- The repo has no test suite. Say so explicitly when reporting verification, and describe what was actually checked.
- Commit and push only when asked, to the branch the session is assigned.
