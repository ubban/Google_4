# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo contains **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs as a single self-contained HTML5 canvas page.

- `index.html` — the entire game: markup, CSS (inline `<style>`), and JavaScript (one inline `<script>` wrapped in an IIFE with `'use strict'`).
- `README.md` — minimal project readme.

There is no build step, no package manager, no bundler, no test suite, and no external dependencies. Keep it that way unless the user asks otherwise.

## Running the game

Open `index.html` directly in a browser. No server is required.

## Code layout inside `index.html`

Rough order of the inline script (search for the section names):

- **DOM references** — `const xxxEl = document.getElementById(...)` at the top of the IIFE.
- **Storage keys** — `STORAGE_*` constants, all prefixed `skyfighter.` (best score, mute, difficulty, reduced motion).
- **Adaptive performance** — `trackPerf` lowers/raises a `quality` level (0–2) based on average frame time.
- **Tuning data** — `DIFFICULTIES` (spawn rates, enemy speed/shoot chance, lives, bombs, boss HP multiplier), `KILLS_PER_STAGE`, `THEMES` (per-stage sky palettes), `ENEMY_TYPES`, `BOSSES`.
- **Audio** — `ensureAudio`, `beep`, `noiseBurst`, `EXPLODE_SFX`. Audio is synthesized with WebAudio; there are no audio files.
- **HUD / effects** — `renderHearts`, `renderBombs`, `flashHit`, `showComboPop`, `showStageBanner`, `screenShake`.
- **Input** — keyboard (`keys` map), mouse (canvas move/click), touch joystick and FIRE/BOMB buttons.
- **Game state** — `resetGame`, `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`, `applyPowerUp`, `fireWeapon`, `useBomb`, `addScore`/`breakChain` (kill-chain multiplier).
- **Boss patterns** — `BOSS_PATTERNS` and the `fire*` helpers (radial, spread, columns, spiral, cross, homing, double ring), driven by `updateBoss`.
- **Update / draw** — `update(dt)` for simulation, `draw()` for rendering, `drawPlane` / `drawBossShip` / `drawClouds` for sprites.
- **Lifecycle** — `loseLife`, `killBoss`, `endGame`, `togglePause`, and the `loop(now)` requestAnimationFrame driver. `dt` is clamped to 0.05 s.

Canvas is sized against a fixed logical resolution (`BASE_W = 480`, `BASE_H = 720`) in `resize()`.

## Controls (keep the on-screen hint in sync)

The hint lives in `#controlsHint` in the overlay. If you change a binding, update that text too.

- Mouse: move to steer, click to fire.
- Arrows / WASD: move. Space: fire.
- X or Shift: bomb. P or Esc: pause.
- Touch: drag left zone to fly, FIRE and BOMB buttons on the right.

## Conventions

- Keep everything in `index.html` unless the user asks for a split. Do not add frameworks or CDN dependencies.
- Use `const`/`let`, arrow functions, and the existing naming style (camelCase functions, UPPER_SNAKE for constant tables).
- Put gameplay numbers in the existing tables (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `THEMES`) rather than hard-coding them in logic.
- Any new `localStorage` key must use the `skyfighter.` prefix and be wrapped so the game still works if storage is unavailable.
- Respect the reduced-motion setting: effects like screen shake, hit flash, and the low-health vignette pulse should be gated on `reducedMotion`.
- Keep the UI accessible: announce important state changes through `announce()` (the `#srLive` region).
- Preserve mobile support: new controls need touch equivalents, and the layout must still work at phone widths (the page uses a fixed-viewport canvas with touch overlays).

## Verification

There are no automated tests. After changing gameplay or UI:

1. Open `index.html` in a browser and start a game.
2. Exercise the change directly (e.g., fire, bomb, pause, difficulty switch, boss encounter).
3. Check the browser console for errors.
4. For layout changes, check both a desktop viewport and a narrow phone-sized viewport.

If you cannot run a browser, say so explicitly in your summary rather than claiming the change was tested.

## Git

- Develop on the branch the task specifies (currently `claude/gifted-tesla-q9pudc` for this session). Do not push to other branches without permission.
- Write commit messages that describe the gameplay or code change, not just "update file".
- Do not open pull requests unless explicitly asked.
