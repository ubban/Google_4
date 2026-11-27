# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo contains **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs in the browser. The whole game is one self-contained file:

- `index.html` (~1,670 lines): inline `<style>`, markup for the HUD/overlays/touch controls, and one inline `<script>` IIFE containing all game logic, rendering, audio, and input.
- `README.md`: a placeholder title only.

There is no build step, no package manager, no bundler, and no test suite. Do not add one unless asked.

## Running and verifying

- Open `index.html` directly in a browser, or serve the folder with any static server (e.g. `python3 -m http.server`).
- There are no automated tests or linters configured. Verify changes by playing the game in a browser: start a run, check movement, firing, bombs, power-ups, boss fights, the pause/mute/motion buttons, and the game-over overlay.
- For touch behaviour, test with a mobile viewport or device emulation. The game uses `touch-action: none` and fixed-aspect scaling.

## Code layout inside `index.html`

The script is organised by comment banners and sections, roughly in this order:

1. **DOM references**: `const xxxEl = document.getElementById(...)`. IDs in the markup must match these exactly.
2. **Storage keys**: `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion`. Keep these stable so players' saved settings survive.
3. **Adaptive performance** (`trackPerf`): steps a `quality` level (0 full / 1 medium / 2 low) based on the average frame time over 45 frames.
4. **Tuning tables** (the main place to balance the game):
   - `DIFFICULTIES`: spawn rate, enemy speed, enemy fire chance, lives, bombs, boss HP multiplier.
   - `KILLS_PER_STAGE`: kills needed to advance a stage.
   - `THEMES`: sky gradients and palettes, cycled per stage.
   - `ENEMY_TYPES`: `grunt`, `interceptor` (weaving), `tank` (multi-hit). Each has size, HP, speed, colours, score, and `minStage`.
   - `BOSSES`: six bosses (VANGUARD through TITAN), each with colours and an `hpMul`.
   - `BOSS_PATTERNS`: bullet-pattern functions (radial, aimed spread, columns, spiral, cross, seeded homing, double ring) used by `updateBoss`.
5. **Audio**: a lazily created `AudioContext` (`ensureAudio`) with `beep` and `noiseBurst` helpers. Sound effects are keyed in `EXPLODE_SFX`. Audio must start from a user gesture.
6. **HUD helpers**: hearts, bombs, combo pops, stage banner, screen shake, flash, and vignette.
7. **Input**: keyboard (arrows/WASD to move, Space to fire, X or Shift to bomb, P or Escape to pause), a virtual joystick and fire/bomb buttons for touch, and mouse move/press on the canvas.
8. **Game state**: `resetGame`, `update(dt)` (the main simulation step), `fireWeapon`, `useBomb`, `applyPowerUp`, `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`, `addScore`/combo handling.
9. **Rendering**: `drawPlane`, `drawBossShip`, and the draw loop. The canvas is a fixed logical size of **480×720** (`BASE_W`, `BASE_H`), scaled by `resize()`.

Weapons are `vulcan` and `laser`. Power-ups are `life`, `bomb`, and `weapon` (toggles the weapon). Player state includes hearts, bombs, weapon level, and a score multiplier that resets when the combo chain breaks.

## Conventions

- Keep the game as a single self-contained HTML file unless the user asks for a split. Do not introduce external CDN scripts or fonts without a reason.
- Keep tuning values in the existing constant tables rather than hard-coding them in logic.
- Preserve accessibility hooks: the `srLive` live region (`announce`) for screen-reader status, and the `reduced-motion` body class / `reducedMotion` setting. That setting scales down screen shake, and the CSS under `body.reduced-motion` turns off flashes, pulses, and banner animations. Honour it in any new effect.
- Keep changes in the existing style: 2-space indent, single quotes, `const`/`let`, and small named functions.
- Use the existing `localStorage` keys. Wrap any new storage access so the game still works if storage is unavailable (private mode, blocked site data).
- Don't break the `'use strict'` IIFE by leaking globals. Only expose what the HTML inline handlers need.

## Git workflow

- Default development branch for this session: `claude/gifted-tesla-hfmm0s`. Commit and push there; don't push to other branches without permission.
- Don't open a pull request unless the user asks for one.
- Commit messages should describe the gameplay or code change in plain words.
