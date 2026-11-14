# CLAUDE.md

Guidance for Claude Code when working in this repository.

## What this repo is

A small, dependency-free browser game, **Sky Fighter** (a Raiden-style vertical shoot-'em-up). The entire game is a single self-contained file:

- `index.html` (~1,670 lines): inline `<style>`, the HTML/UI markup, and one inline `<script>` IIFE containing all game logic, rendering, and audio.
- `README.md`: a placeholder. Its only content is a stray timestamp line. Do not treat it as documentation.

There is no build step, package manager, test suite, linter, or bundler. Nothing is installed or compiled.

## Running it

Open `index.html` directly in a browser, or serve the directory with any static server (for example, `python3 -m http.server` from the repo root). No install step is needed.

## Verifying changes

There are no automated tests. For changes to gameplay, rendering, or UI:

1. Open the page in a browser and start a game from the Easy, Normal, and Hard buttons.
2. Check the affected behavior directly: movement, firing, bombs, power-ups, boss fights, pause, game over, and the best-score update.
3. Check the touch layout too if the change touches controls or the layout. Touch controls appear only on touch-capable devices.

If a headless browser is needed, Playwright is available in the cloud environment.

## Code layout inside `index.html`

The script is wrapped in an IIFE with `'use strict'`. It does not export anything or expose globals. Within it, the code is grouped roughly in this order:

| Section | What it covers |
| --- | --- |
| DOM references | `const xEl = document.getElementById(...)` for every element the script touches. Keep IDs in sync with the markup. |
| Storage keys | `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion` in `localStorage`. |
| Config tables | `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES` (sky palettes per stage), `ENEMY_TYPES`, `BOSSES`, and `BOSS_PATTERNS`. |
| Audio | A Web Audio synth: `ensureAudio()`, `beep()`, `noiseBurst()`, and a named `sfx` table (`shoot`, `laser`, `hit`, `bomb`, `powerup`, `gameover`, and others). It has no audio files. |
| Input | Keyboard handlers, a virtual joystick and buttons for touch, and mouse steering and firing on the canvas. |
| Game state | `resetGame()` and the state variables: `player`, `bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`, `boss`. |
| Simulation | `update(dt)` advances the state each frame. `spawnEnemy`, `spawnBoss`, `applyPowerUp`, `useBomb`, `fireWeapon`, `updateBoss`, `killBoss`, and `loseLife` live here. |
| Rendering | `draw()` and helpers `drawPlane`, `drawBossShip`, `drawClouds`. It draws to a fixed 480×720 canvas (`BASE_W` × `BASE_H`) that is scaled to fit the window. |
| Loop and perf | `loop(now)` drives `requestAnimationFrame`. `trackPerf` adapts a `quality` level (0 to 2) from the average frame time. |

## Conventions

- **Keep it one file.** Put new code, styles, and markup in `index.html` unless the user asks for a split. Do not add a build system, frameworks, or dependencies without asking.
- **Keep the style of the surrounding code.** It uses `const`/`let`, arrow functions, two-space indentation, and single quotes in JavaScript. Keep the code in the same IIFE.
- **Time-based updates.** Movement and timers use `dt` (seconds). Do not hard-code per-frame step sizes.
- **Config over magic numbers.** Put tunable values (difficulty, enemy stats, boss HP multipliers, spawn rates) in the existing config tables rather than inline.
- **Accessibility hooks are deliberate.** Keep the `announce()` / `srLive` live region, the `reduced-motion` body class, and the `prefers-reduced-motion` default. Screen shake and flash effects must respect the reduced-motion setting.
- **Persistence is best-effort.** Writes to `localStorage` use the `skyfighter.*` keys. Do not rename existing keys without a migration, or players lose their saved best score and settings.
- **Adding enemies, bosses, or power-ups** means extending the relevant config table (`ENEMY_TYPES`, `BOSSES`, `BOSS_PATTERNS`) and the matching spawn, update, and draw paths. Check `applyPowerUp` and the `powerUps` spawn logic when adding a pickup type.

## Controls (for reference when testing)

- **Keyboard:** Arrow keys or WASD to move, Space to fire, X or Shift for a bomb, P or Esc to pause.
- **Mouse:** move to steer, click to fire.
- **Touch:** drag on the left to fly, tap FIRE, tap BOMB.

## Git

- Development branch: `claude/gifted-tesla-389jnf`. Commit and push there. Do not push to other branches without explicit permission.
- Do not open a pull request unless explicitly asked.
