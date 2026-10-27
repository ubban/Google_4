# CLAUDE.md

Guidance for Claude Code when working in this repository.

## What this repo is

A single-page browser game, **Sky Fighter**: a Raiden-style vertical shoot-'em-up drawn on a `<canvas>`. The whole game lives in `index.html`:

- `<style>` (around lines 10–359): all CSS, including HUD, overlays, and touch controls.
- `<body>` (from line 361): the canvas and the HUD/overlay markup.
- One inline `<script>` (around lines 422–1667) wrapped in an IIFE. It holds all game state, logic, rendering, and audio.

`README.md` is a stub. There is no build step, package manager, test suite, or linter. Nothing is installed.

## Running and checking changes

- Open `index.html` directly in a browser. No server is needed.
- There are no automated tests. Verify changes by playing: start a run, check the affected behavior, and test on both desktop and a touch-sized viewport.
- Keep the game working from a plain `file://` open. Don't add anything that requires a server, a bundler, or network access at runtime.

## Code layout inside the script

The script is organized in sections. Keep new code in the matching section:

1. **DOM refs and constants**: `getElementById` lookups, `STORAGE_*` keys, `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES`, `ENEMY_TYPES`, `BOSSES`.
2. **Persisted settings**: `bestScore`, `muted`, `difficulty`, `reducedMotion`, all read from `localStorage` at startup.
3. **Audio**: a lazily created `AudioContext` (`ensureAudio`), with `beep` and `noiseBurst` primitives and a named `sfx` table.
4. **HUD and feedback**: hearts/bombs rendering, flash, combo pop, stage banner, screen shake, `announce` for screen readers.
5. **Responsive scaling**: `resize()` fits a fixed `BASE_W × BASE_H` (480 × 720) play area into the window, capped at 1.6×.
6. **Input**: `keys` (keyed by `e.code`), a virtual joystick and fire/bomb buttons for touch, and mouse aim on the canvas.
7. **Game state**: `player`, `bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`, `boss`, plus score, lives, bombs, stage, and chain/multiplier.
8. **Entities and spawning**: `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`.
9. **Update**: `update(dt)` runs the simulation. `updateBoss(dt)` runs boss phases.
10. **Boss attack patterns**: `BOSS_PATTERNS`, indexed by boss design, built from `fire*` helpers.
11. **Rendering**: `draw()` plus `drawPlane`, `drawBossShip`, `drawClouds`.
12. **Main loop**: `loop(now)` clamps `dt` to 0.05 s, calls `trackPerf`, then `update` and `draw`.

## Game rules worth knowing

- **Difficulty** (`DIFFICULTIES`) sets spawn cadence, enemy speed, enemy fire chance, starting lives and bombs, and a boss HP multiplier. The choice is saved to `localStorage`.
- **Stages**: a stage ends after `KILLS_PER_STAGE` (18) kills, and then a boss spawns. Defeating the boss advances the stage. Stage `n` uses `THEMES[(n-1) % 6]` for the sky and `BOSSES[(n-1) % 6]` for the boss.
- **Enemies** (`ENEMY_TYPES`): `grunt` (stage 1+), `interceptor` (stage 1+, weaves), `tank` (stage 2+, 4 HP).
- **Power-ups** drop from destroyed enemies: life (6%), bomb (8%), weapon (20%). Weapon power-ups raise weapon level to 3. After that they toggle between VULCAN and LASER at level 1.
- **Score** is multiplied by a chain multiplier: `1 + min(9, floor(chain / 5))`, so it reaches ×10. Each kill extends the chain, and the chain breaks (`breakChain()`) if no kill lands within about 2.2 s.
- **Lives and bombs** are capped by `maxLives` and `maxBombs`.

## Controls

- Keyboard: arrows or WASD to move, Space to fire, X or Shift to bomb, P or Esc to pause.
- Mouse: move to aim on the canvas, hold to fire.
- Touch: joystick on the left, fire and bomb buttons on the right.
- Mute, motion, and pause are also buttons in the HUD (`muteBtn`, `motionBtn`, `pauseBtn`).

## Persistence

Only `localStorage` is used, and every key is prefixed `skyfighter.`:

- `skyfighter.bestScore`
- `skyfighter.muted`
- `skyfighter.difficulty`
- `skyfighter.reducedMotion`

Any new persisted setting should use this prefix and add its key next to the existing `STORAGE_*` constants.

## Conventions

- **Single file**: keep everything in `index.html`. Don't split out modules or add a build step unless the user asks.
- **Plain JavaScript**: no frameworks, no transpilation, and no ES features that break in current evergreen browsers.
- **Match local style**: 2-space indentation, `const` by default, short helper functions, and comments only where the logic isn't obvious.
- **Honor reduced motion**: visual effects like screen shake and flashes should check the `reducedMotion` flag, as the existing code does.
- **Respect the quality tier**: `quality` (0 full, 1 medium, 2 low) is set by `trackPerf` from frame timing. New particle or glow effects should scale with it, as `spawnExplosion` does.
- **Accessibility**: use `announce()` for important state changes that a screen reader should hear. Keep the `aria-*` attributes in the markup intact.
- **Keep the frame rate in mind**: avoid per-frame allocations in `update` and `draw` where a reused value would do.

## Working in this repo

- The default branch is `main`. Feature work goes on the branch you were given.
- Make small, focused commits with descriptive messages.
- Test a change in the browser before committing. Mention in the commit or summary what you played and checked.
