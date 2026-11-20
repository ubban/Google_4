# CLAUDE.md

Guidance for working in this repository.

## Project overview

`Google_4` is a small, dependency-free browser game, **Sky Fighter**: a Raiden-style vertical shoot-'em-up. The entire game lives in one file, `index.html` (~1,670 lines), with inline CSS in `<style>` and inline JavaScript in a single `<script>` block wrapped in an IIFE. `README.md` is a stub.

There is no build step, no package manager, no bundler, and no test suite. Do not add one unless asked.

## Running it

Open `index.html` directly in a browser, or serve the folder with any static server, for example:

```bash
python3 -m http.server 8000   # then visit http://localhost:8000
```

## Repository layout

| Path         | Purpose                                                  |
|--------------|----------------------------------------------------------|
| `index.html` | The whole game: markup, styles, and logic                |
| `README.md`  | Project title stub                                       |
| `CLAUDE.md`  | This file                                                |

## Code structure of `index.html`

The script is one IIFE with `'use strict'`. Its sections, in file order:

- **DOM refs and constants**: element lookups, `STORAGE_*` localStorage keys (`skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion`).
- **Config tables**: `DIFFICULTIES` (easy / normal / hard: spawn rates, enemy speed and fire chance, lives, bombs, boss HP multiplier), `KILLS_PER_STAGE` (18, which triggers the boss warning), `THEMES` (six stage sky palettes, indexed by `(stage - 1) % 6`), `ENEMY_TYPES` (grunt, interceptor, tank), and `BOSSES` (six bosses, each with its own colors and HP multiplier).
- **Adaptive quality**: `trackPerf` samples frame times over 45 frames and steps `quality` between 0 (full), 1 (medium), and 2 (low) to protect frame rate.
- **Audio**: a Web Audio synth (`ensureAudio`, `beep`, `noiseBurst`, `panFor`). Sounds are generated, not loaded from files. `ensureAudio` must run from a user gesture.
- **HUD and effects**: hearts, bombs, combo pop, stage banner, screen shake, flash, and an `announce()` helper that writes to an ARIA live region.
- **Layout**: a fixed logical canvas of `BASE_W = 480` × `BASE_H = 720`, scaled to fit by `resize()`.
- **Input**: keyboard (`keys` map keyed by `e.code`), mouse/pointer via `canvasPointFromEvent`, and touch (a virtual joystick in `moveZone`, plus `fireBtn` and `bombBtn`). Touch controls are shown only when `isTouchDevice` is true.
- **Simulation**: `resetGame`, `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`, `applyPowerUp`, `fireWeapon`, `useBomb`, `addScore` (score multiplier with `breakChain`), `loseLife`, `endGame`, `killBoss`, and `togglePause`. The per-frame step is `update(dt)`.
- **Boss attacks**: `BOSS_PATTERNS` plus the `fireRadial`, `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`, `fireCross`, `fireHomingSeeded`, and `fireDoubleRing` helpers. `updateBoss(dt)` drives them.
- **Rendering**: `draw()` renders each frame, with helpers `drawPlane`, `drawBossShip`, and `drawClouds`.
- **Main loop**: `loop(now)` clamps `dt` to 0.05 s, calls `trackPerf`, then `update` and `draw` while the game is running and not paused. It runs through `requestAnimationFrame`.

## Controls

- Mouse: move to steer, click to fire.
- Arrow keys or WASD: move. Space: fire.
- X or Shift: bomb. P or Esc: pause.
- Touch: drag the left zone to fly, tap FIRE, tap BOMB.

Keep the controls hint (`#controlsHint` in the markup) in sync with any binding change.

## Persistence

Only the four `skyfighter.*` keys above are written to `localStorage`. Best score is saved at game over. Every localStorage read and write should tolerate being unavailable, for example in private browsing, because the game is expected to run without it.

## Conventions

- **Keep it a single file.** New game code goes in the existing `<script>` IIFE, and new styles go in the existing `<style>` block, unless the user asks for a split.
- **No dependencies.** Do not add libraries, CDN links, or a build toolchain without asking.
- **Preserve the IIFE and `'use strict'`.** Nothing should leak to `window` except what already does.
- **Match the existing style**: two-space indentation, `const`/`let`, arrow functions for callbacks, and section banner comments of the form `// ---------- Name ----------`.
- **Use `dt` for movement and timers** so behavior is frame-rate independent. Clamp it as `loop` already does.
- **Tune with the config tables.** Prefer changing `DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, or `THEMES` over hard-coding numbers in logic.
- **Accessibility**: respect `prefers-reduced-motion` (the `reducedMotion` flag and the `reduced-motion` body class), and announce important state changes through `announce()`.
- **Mobile**: keep the `viewport-fit=cover` setting in the `<meta viewport>` tag and the `env(safe-area-inset-*)` offsets on the HUD intact.

## Verifying changes

There are no automated tests. After a change, check it by hand in a browser:

1. Load the page with no console errors.
2. Start a game and check movement, firing, bombs, and pause (P and Esc).
3. Get 18 kills to trigger the boss warning, then defeat the boss and check the stage banner and theme change.
4. Check the boss HP bar and attack patterns during the fight.
5. Lose all lives and check the game-over overlay and the best-score update.
6. Toggle mute, difficulty, and reduced motion, then reload to confirm they persist.
7. If the change touches layout or input, check it at phone width with touch controls.

For UI changes, a screenshot at a realistic viewport is the most reliable check.

## Git

- Development branch for automated work: `claude/gifted-tesla-m9jq9m`.
- Write clear, present-tense commit messages that say what changed and why.
- Do not open pull requests unless asked.
