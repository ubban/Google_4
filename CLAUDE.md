# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

**Sky Fighter** is a single-file, Raiden-style vertical shoot-'em-up that runs in the browser. There is no build step, no package manager, and no test suite.

- `index.html` — the entire game: inline `<style>`, markup, and one inline `<script>` IIFE (`'use strict'`).
- `README.md` — placeholder; currently only a heading.

To play, open `index.html` directly in a browser (or serve the folder with any static file server).

## Layout of `index.html`

- **Styles (~lines 10–360):** dark `color-scheme`, HUD pills (score, best, multiplier, weapon, hearts, bombs), overlay/menu, boss bar, vignette/flash effects, and touch-control styling.
- **Markup (~lines 360–420):** the `<canvas id="gameCanvas">`, HUD elements, overlay with difficulty buttons (`.diffBtn`), touch controls (joystick `moveZone`, `fireBtn`, `bombBtn`), and an ARIA live region (`srLive`).
- **Script (~lines 422–1670):** all game logic, grouped roughly as:
  - DOM refs and `STORAGE_*` keys (persistence via `localStorage`).
  - Config tables: `DIFFICULTIES`, `THEMES` (stage backgrounds), `ENEMY_TYPES`, `BOSSES`, `BOSS_PATTERNS`.
  - Audio: a WebAudio `sfx` object built from `beep()` / `noiseBurst()`; created lazily by `ensureAudio()` on first user gesture.
  - Input: keyboard (`keys[e.code]`), mouse (`canvasPointFromEvent`), and touch joystick/buttons.
  - Simulation: `resetGame`, `update(dt)`, `spawnEnemy`, `spawnBoss`, `fireWeapon`, `useBomb`, `applyPowerUp`, `updateBoss`, `killBoss`, `loseLife`, `endGame`, `togglePause`.
  - Rendering: `draw()`, `drawPlane`, `drawBossShip`, `drawClouds`.
  - Main loop: `loop(now)` drives fixed-ish `dt` updates and `draw()`.

## Game rules (for reference when changing balance)

- **Difficulty** (`DIFFICULTIES`): easy / normal / hard change spawn rate, enemy speed, enemy shot chance, starting lives, starting bombs, and boss HP multiplier.
- **Stages:** `KILLS_PER_STAGE` (18) kills per stage. Each stage ends in a boss from `BOSSES`, cycling by `(stage - 1) % BOSSES.length`. Background theme cycles through `THEMES`.
- **Enemies:** `grunt`, `interceptor` (fast, weaving), and `tank` (4 HP, from stage 2).
- **Weapons:** `vulcan` (spread shots) and `laser` (piercing). Weapon power-ups raise the level from 1 to 3; past level 3 they swap weapon type and reset to level 1.
- **Power-ups:** `life` (6%), `bomb` (8%), and `weapon` (20%) chance drops from destroyed enemies (see `maybeSpawnPowerUp`).
- **Score / combo:** `addScore` multiplier is `1 + min(9, floor(chain / 5))`. The chain resets via `breakChain` when its timer runs out.
- **Bombs:** `useBomb` clears all enemy bullets, destroys every on-screen enemy (5 points each), and deals 24 damage to the boss.
- **Controls:** arrows or WASD to move; Space or hold the fire button to shoot; X / Shift for a bomb; P or Escape to pause. Touch uses the joystick and on-screen fire/bomb buttons.
- **Persistence (`localStorage`):** `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion`.
- **Accessibility:** reduced-motion mode (defaults to `prefers-reduced-motion`, overridable via the toggle) scales screen shake down and hides flash/vignette/banner effects via the `body.reduced-motion` class; `announce()` writes to the live region.
- **Performance:** `trackPerf` adapts a `quality` level (0–2) from frame timings to reduce effects on slow devices.

## Conventions

- Keep the game as a single self-contained `index.html` unless the user asks for a restructure. Do not introduce a bundler, framework, or external dependencies without a clear request.
- Keep config values in the existing tables (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `THEMES`) rather than scattering magic numbers through the logic.
- Preserve the `'use strict'` IIFE structure and avoid leaking globals.
- Maintain touch and keyboard parity: any new action needs both a key binding and a touch control.
- Respect `reducedMotion` for any new screen shake, flashes, or animation.
- Keep HUD text and new UI usable at phone width (the viewport is `user-scalable=no`, `touch-action: none`).

## Verifying changes

There are no automated tests or linters configured. To verify a change:

1. Open `index.html` in a browser and play through at least one stage and one boss.
2. Check the browser console for errors.
3. Test pause (P / Escape), mute, difficulty switching, and game-over / best-score persistence after reload.
4. For UI or layout changes, check both desktop and a phone-sized viewport.

## Git

- Work on the branch the session specifies; do not push to other branches without permission.
- Commit messages should describe the gameplay or code change concretely (e.g., "Add stage 7 boss pattern", "Fix bomb not clearing enemy bullets").
