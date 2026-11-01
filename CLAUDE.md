# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a single-file browser game, **Sky Fighter**: a Raiden-style vertical shoot-em-up rendered on an HTML5 `<canvas>`. There is no build step, no package manager, and no test suite.

| File        | Purpose                                                                 |
|-------------|-------------------------------------------------------------------------|
| `index.html` | The entire game: markup (~lines 1–420), CSS (`<style>`, ~lines 10–362), and JavaScript (`<script>`, ~lines 422–1669). |
| `README.md`  | Placeholder title only.                                                 |

## Running the game

No install or build is needed. Open `index.html` directly in a browser, or serve the directory with any static server, for example:

```bash
python3 -m http.server 8000   # then visit http://localhost:8000/index.html
```

## Code layout inside `index.html`

The script is one IIFE (`(function () { 'use strict'; ... })();`). Everything is private to it; nothing is exposed on `window`. Within it, the code is organized roughly as follows:

- **DOM references**: `const xxxEl = document.getElementById(...)` block at the top of the script. If you add UI, add its element reference here.
- **Persistence keys**: `STORAGE_*` constants (`skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion`) used with `localStorage`.
- **Adaptive quality**: `trackPerf()` samples frame time over 45 frames and steps `quality` (0 full, 1 medium, 2 low) to reduce effects on slow devices.
- **Tuning data**: `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES` (per-stage sky palettes), `ENEMY_TYPES`, and `BOSSES`. Balance changes usually belong here rather than in logic.
- **Audio**: `ensureAudio()`, `beep()`, `noiseBurst()`, and the `sfx` object, built on the Web Audio API. Audio starts only after a user gesture, so call `ensureAudio()` from input handlers.
- **Input**: keyboard (`keys` map, `keydown`/`keyup`), mouse on the canvas, and touch controls (a joystick in `#moveZone`, plus `#fireBtn` and `#bombBtn`).
- **Game state and update**: `resetGame()`, then `update(dt)` for the per-frame simulation. Spawning uses `spawnEnemy()`, `spawnBoss()`, and `maybeSpawnPowerUp()`. Weapons use `fireWeapon()`, and bombs use `useBomb()`.
- **Bosses**: `BOSS_PATTERNS` is an array of attack functions (`fireRadial`, `fireAimedSpread`, `fireSpiralBurst`, and so on). `updateBoss()` cycles through them; `killBoss()` advances the stage.
- **Rendering**: `draw()` runs once per frame after `update()`. Helpers include `drawPlane()`, `drawBossShip()`, and `drawClouds()`.
- **Main loop**: `loop(now)` is driven by `requestAnimationFrame`. `dt` is clamped to 50 ms. The loop calls `update` and `draw` only when `running && !paused`.

Game coordinates use a fixed logical canvas of `BASE_W = 480` × `BASE_H = 720`. `resize()` scales that to the viewport, so gameplay code should work in these base units.

## Conventions

- **Single file.** Keep the game in `index.html` unless the user asks for a split. Do not add a bundler, framework, or package manager without a clear reason.
- **Vanilla JS, ES2015+.** Use `const`/`let`, arrow functions, and template literals, matching the existing style. Two-space indentation.
- **Respect reduced motion.** Anything that shakes the screen or flashes should check `reducedMotion` and the `reduced-motion` body class.
- **Keep accessibility hooks.** Announcements go through `announce()` into the `#srLive` live region. Keep labels and `aria-*` attributes on buttons when you change controls.
- **Persist only through the existing keys.** Wrap new `localStorage` reads and writes in the same pattern used for the `STORAGE_*` keys.
- **Keep tuning in data tables.** Prefer changing the `DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, and `THEMES` tables over hard-coding numbers in logic.

## Testing and verification

There is no automated test suite. To verify a change:

1. Open `index.html` in a browser and play through at least one stage.
2. Check all three difficulties (`easy`, `normal`, `hard`) if you changed balance values.
3. Check touch controls with browser device emulation if you changed input or layout.
4. Check the console for errors. A clean run is the baseline.

When you change tuning or logic, say in your summary which of these checks you ran and which you could not.

## Git

- Development branch for this session: `claude/gifted-tesla-2br612`. Base work on `main`.
- Write clear, imperative commit messages that describe the change.
- Do not commit generated or editor files; there are currently none in the repo.
