# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo contains **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs in the browser on an HTML5 canvas. The entire game lives in one file:

- `index.html` (~1,670 lines): markup, CSS (`<style>`, lines ~10–362), canvas and HUD/overlay elements, and all game logic in a single IIFE `<script>` (starts ~line 422).
- `README.md`: a one-line placeholder. Keep it accurate if you expand it.

There is no build step, no package manager, no bundler, no framework, and no test suite. Do not add `package.json`, a build pipeline, or dependencies unless the user asks for them.

## Running the game

Open `index.html` directly in a browser, or serve the directory with any static server, for example:

```bash
python3 -m http.server 8000   # then visit http://localhost:8000
```

Verify changes by playing the game in a browser. There are no automated tests or linters configured, so a manual check is the only verification unless you add one at the user's request.

## Code layout inside `index.html`

The game script is organized by section comments. Key pieces, roughly in file order:

- **DOM references and storage keys**: element lookups at the top of the IIFE. `STORAGE_*` constants use the `skyfighter.*` prefix for `localStorage` (best score, mute, difficulty, reduced motion).
- **Adaptive performance** (`trackPerf`): samples frame times and steps a `quality` level (0 full, 1 medium, 2 low) up or down. Draw code should respect this.
- **Tuning tables**: `DIFFICULTIES` (spawn rates, enemy speed and fire chance, lives, bombs, boss HP multiplier), `KILLS_PER_STAGE`, `THEMES` (sky palettes per stage), `ENEMY_TYPES` (grunt, interceptor, tank), and `BOSSES` (six bosses, cycled by stage). Balance changes usually belong here, not in logic.
- **Audio**: `ensureAudio`, `beep`, `noiseBurst`, and named SFX. The `AudioContext` is created lazily on the first user gesture, so keep audio creation behind user input.
- **Input**: keyboard (`keys` map by `e.code`), touch joystick and fire/bomb buttons, and pointer handling via `canvasPointFromEvent`. Current bindings: arrows/WASD move, Space fires, `X`/Shift bomb, `P`/Escape pause.
- **Game state and flow**: `resetGame`, `update(dt)`, `applyPowerUp`, `useBomb`, `fireWeapon`, `loseLife`, `endGame`, `togglePause`. `update` is the main simulation step.
- **Boss system**: `spawnBoss`, `BOSS_PATTERNS` (bullet-pattern functions such as radial, spiral, cross, homing), `updateBoss`, `killBoss`.
- **Rendering**: `draw()` plus helpers `drawPlane`, `drawBossShip`, `drawClouds`. Drawing is pure canvas 2D.
- **Main loop**: `loop(now)` clamps `dt` to 50 ms, calls `trackPerf`, then `update` and `draw` while running and not paused.

## Conventions

- **Single file, vanilla JS.** Keep the existing style: `'use strict'`, `const`/`let`, arrow functions for callbacks, no modules, no external libraries. No CDN scripts unless the user asks.
- **Keep the IIFE intact.** Game state is closure-scoped; do not leak globals. Only the `localStorage` keys are persistent state.
- **Tune through constants.** Put numbers you may want to adjust in the tuning tables above rather than inline magic values.
- **Respect reduced motion.** When adding screen shake, flashes, or particle-heavy effects, gate them on the `reducedMotion` flag and the `quality` level.
- **Keep accessibility hooks.** Status changes go through `announce()` for screen readers. Preserve ARIA attributes and the `reduced-motion` body class when editing markup.
- **Mobile and desktop both matter.** The layout uses viewport meta, `resize()`, and touch controls. Test any HUD or input change at phone width as well as desktop.
- **Pause-aware timing.** Gameplay timers use `dt` and should freeze while `paused` or `!running`.

## Working in this repo

- Make focused edits. Because everything is in one file, avoid reformatting unrelated sections so diffs stay reviewable.
- After changing game logic, play through at least one stage and check the affected behavior in a browser (start, fire, collect power-up, bomb, boss if relevant, game over, restart).
- Git: develop on the branch assigned to the session. Commit with clear messages and push with `git push -u origin <branch>`. Do not open pull requests unless the user asks.

## Out of scope unless requested

- Introducing a build system, frameworks, TypeScript, or a test harness.
- Splitting `index.html` into multiple files. If you do this, update this document to match.
