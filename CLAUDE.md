# CLAUDE.md

Guidance for Claude Code when working in this repository.

## What this repo is

`Google_4` is a small static web project. Its only real content is **Sky Fighter**, a
Raiden-style vertical shoot-'em-up that runs in a browser canvas. There is no build step,
no package manager, no framework, and no test suite.

Files:

- `index.html` (~1,670 lines): the entire game. CSS is in a `<style>` block (about lines 10–359),
  markup in the body (about lines 360–421), and all JavaScript in one `<script>` block
  (about lines 422–1667), wrapped in a `(function () { 'use strict'; … })();` IIFE.
- `README.md`: a placeholder title only.

## Running it

Open `index.html` directly in a browser, or serve the folder with any static server, for example:

```bash
python3 -m http.server 8000   # then visit http://localhost:8000
```

There is nothing to install or compile.

## Architecture of `index.html`

All script state and functions live inside the IIFE, so they are not globals. Major sections, in file order:

- **Storage keys**: `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`,
  `skyfighter.reducedMotion`. Persisted via `localStorage`.
- **Adaptive performance** (`trackPerf`): rolling average of frame times drives a `quality` level
  (0 full, 1 medium, 2 low).
- **Tuning tables**: `DIFFICULTIES` (spawn rates, enemy speed and fire chance, lives, bombs, boss HP
  multiplier), `THEMES` (six background palettes cycled per stage), `ENEMY_TYPES`
  (`grunt`, `interceptor`, `tank`), and `BOSSES` (six bosses, indexed by `(stage - 1) % 6`). `KILLS_PER_STAGE` is 18.
- **Audio**: Web Audio API, built lazily by `ensureAudio()`. Sound effects come from `beep()` and
  `noiseBurst()`. Muting is persisted.
- **HUD**: DOM elements updated by helpers such as `renderHearts`, `renderBombs`, `showStageBanner`,
  `showComboPop`. A hidden `srLive` element carries screen-reader announcements via `announce()`.
- **Layout**: fixed logical size `BASE_W = 480`, `BASE_H = 720`, scaled to fit the viewport in `resize()`.
- **Input**:
  - Keyboard: arrows or WASD to move; Space or hold to fire; `X`, `Shift` for a bomb;
    `P` or `Escape` to pause.
  - Touch: virtual joystick (`moveZone`), fire button, bomb button.
  - Mouse: `canvas` mouse-move and mousedown control.
- **Game state**: `let` variables scoped to the IIFE (`player`, `bullets`, `enemies`, `enemyBullets`,
  `powerUps`, `boss`, `score`, `lives`, `stage`, `chain`, `multiplier`, …). `resetGame()` reinitialises them.
- **Simulation**: `update(dt)` advances everything each frame. It handles movement, firing,
  spawning, collisions, power-up pickup, and boss patterns. Boss attack patterns are small
  functions such as `fireRadial`, `fireAimedSpread`, `fireSpiralBurst`, `fireHomingSeeded`.
- **Rendering**: `draw()` plus `drawPlane`, `drawBossShip`, and related helpers, using canvas 2D.
- **Main loop**: `loop(now)` clamps `dt` to 0.05 s, calls `trackPerf`, then `update` and `draw`
  when `running && !paused`, and reschedules with `requestAnimationFrame`.

Power-ups are `life`, `bomb`, and `weapon`. The weapon power-up first raises `player.level` (up to 3),
then toggles between `vulcan` and `laser`.

## Conventions

- Keep the game in the single `index.html` file unless there is a clear reason to split it. Do not
  introduce a bundler or framework without asking.
- Tuning values live in the constant tables near the top of the script (`DIFFICULTIES`, `ENEMY_TYPES`,
  `BOSSES`, `THEMES`). Prefer adjusting those over hard-coding new numbers into `update()`.
- Keep the storage keys namespaced with `skyfighter.`.
- Respect `reducedMotion` for any new screen shake or flashing effect, and keep audio gated behind
  `ensureAudio()` so it only starts after a user gesture.
- Keep layout working at phone width: the page is touch-first, uses `touch-action: none`, and
  scales the canvas to fit.
- Use plain modern browser JavaScript, matching the existing code (arrow functions, `const`/`let`).
  No transpiling is set up.

## Verifying changes

There are no automated tests or linters. To check a change:

1. Open `index.html` in a browser and play through at least one stage.
2. Check keyboard, and if relevant touch or mouse, input.
3. Check the console for errors.
4. For layout or performance work, test at a phone-sized viewport as well as desktop.

Playwright is available in some environments (`require('playwright')`) and can drive the page
headlessly for smoke tests if needed.

## Git

- Default development branch for this session: `claude/gifted-tesla-mqj1xt`. Push there with
  `git push -u origin <branch>`.
- Do not open pull requests unless the user asks.
