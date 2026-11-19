# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo contains **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs in the browser. The whole game is one self-contained file:

- `index.html` (~1,670 lines): HTML markup, inline `<style>`, and one inline `<script>` IIFE. No build step, no bundler, no external dependencies, no network calls.
- `README.md`: placeholder title only.

To play, open `index.html` directly in a browser. Nothing needs to be installed.

## Repository layout

| Path | Purpose |
| --- | --- |
| `index.html` | The entire game (markup, CSS, JS) |
| `README.md` | Minimal project readme |
| `CLAUDE.md` | This file |

There is no `package.json`, test suite, linter config, or CI. Verification is manual: load the page in a browser and play.

## Architecture of `index.html`

The file is organized top to bottom as:

1. **`<style>`**: layout, HUD, overlay, touch controls, and `body.reduced-motion` overrides.
2. **Markup**: `<canvas id="gameCanvas">`, HUD elements (score, best, multiplier, weapon, hearts, bombs), overlays, touch controls (`#moveZone`, `#fireBtn`, `#bombBtn`), and a hidden `aria-live` region (`#srLive`).
3. **`<script>` IIFE** (`'use strict'`), with these sections in order:
   - DOM lookups and `STORAGE_*` localStorage keys (`skyfighter.bestScore`, `.muted`, `.difficulty`, `.reducedMotion`)
   - Adaptive quality: `trackPerf` drops the render quality level (0 full, 1 medium, 2 low) when frame time is high
   - Config tables: `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES`, `ENEMY_TYPES`, `BOSSES`, `BOSS_PATTERNS`
   - Audio: `ensureAudio`, `beep`, `noiseBurst`, and the `sfx` helpers (Web Audio, created lazily on first user gesture)
   - HUD and feedback helpers: hearts, bombs, flashes, combo pop, stage banner, screen shake
   - Input: keyboard, joystick and fire/bomb buttons for touch, mouse steering and firing on the canvas
   - Game state and logic: `resetGame`, `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`, `applyPowerUp`, `useBomb`, `fireWeapon`, `update(dt)`, `updateBoss`, `killBoss`, `loseLife`, `endGame`, `togglePause`
   - Rendering: `drawPlane`, `drawBossShip`, `drawClouds`, `draw()`
   - Main loop: `loop(now)` calls `trackPerf`, then `update(dt)` and `draw()` while running and not paused. `dt` is clamped to 0.05 s.

Everything is state held in closure-scoped `let` variables. There are no modules, classes, or exported symbols.

## Game configuration

Tune gameplay by editing these tables. Prefer changing values here over changing logic.

- **`DIFFICULTIES`** (`easy` / `normal` / `hard`): `spawnBase`, `spawnMin`, `enemySpeed`, `enemyShootChance`, `lives`, `bombs`, `bossHpMul`.
- **`KILLS_PER_STAGE`**: kills needed to advance a stage (currently 18).
- **`THEMES`**: six sky palettes, cycled by stage. Each has `sky` gradient stops, `cloud` and `sun` RGB strings, and a display `name`.
- **`ENEMY_TYPES`**: `grunt`, `interceptor` (weaving), `tank` (4 HP, unlocks at stage 2). Each has size, `hp`, `speedMul`, colors, `score`, and `minStage`.
- **`BOSSES`**: six bosses (VANGUARD through TITAN), each with colors, `hpMul`, and a name shown in the boss bar. The boss for a stage is `BOSSES[(stage - 1) % BOSSES.length]`. `BOSS_PATTERNS` are the attack functions (radial, spread, columns, spiral, cross, homing, double ring), looked up per boss by index.

## Controls

| Input | Action |
| --- | --- |
| Mouse | Move to steer; click to fire |
| Arrows / WASD | Move |
| Space | Fire (held) |
| X or Shift | Bomb |
| P or Esc | Pause |
| Touch | Drag in the left zone to fly; tap FIRE; tap BOMB |

The on-screen hint text (`#controlsHint`) must stay in sync with these bindings.

## Working conventions

- **Single-file constraint.** Keep the game in `index.html`. Do not split out JS or CSS files, add a bundler, or add dependencies unless the user asks for it.
- **No frameworks.** Use vanilla DOM and Canvas 2D. Keep new code in the existing IIFE; do not add globals.
- **Persistence.** Only the four `skyfighter.*` localStorage keys are used. Wrap any new storage access so the game still works when storage is unavailable (private mode, blocked site data).
- **Accessibility.** Respect `prefers-reduced-motion` (the `reducedMotion` flag and `body.reduced-motion` class) when adding screen shake, flashes, or other motion. Announce important state changes through `announce()` into `#srLive`.
- **Performance.** The render loop runs every frame. Avoid per-frame allocations in hot paths, and respect the adaptive `quality` level when adding expensive effects.
- **Audio.** Create the `AudioContext` only from a user gesture (`ensureAudio()`), since browsers block autoplay.
- **Mobile.** Keep the layout working for touch and for resize/orientation changes (`resize()` scales a `480×720` logical canvas).
- **Keep it small.** Match the existing style: two-space indentation, `const`/`let`, arrow functions for callbacks, short section comments with `// ---------- Name ----------`.

## Verification

No automated tests exist. Before reporting a change as working:

1. Open `index.html` in a browser (or serve the directory with any static server).
2. Check the browser console for errors on load and during play.
3. Exercise the change: start a game, switch difficulty, reach a stage change and a boss, use bombs and power-ups, pause and resume, and check mute and reduced-motion toggles.
4. For layout changes, check both a desktop viewport and a phone-sized viewport with touch controls.

State in the final report what was actually exercised and what was not.

## Git

- Work on the branch the session specifies; do not push to other branches without permission.
- Keep commits focused. Write messages that describe the gameplay or code change, not the file mechanics.
- Do not create pull requests unless the user asks for one.
