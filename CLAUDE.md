# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo is **Sky Fighter**, a single-file, Raiden-style vertical shoot-'em-up that runs in the browser on an HTML `<canvas>`. There is no build step, package manager, framework, or test suite.

Files:

- `index.html` (~1670 lines): the entire game. Markup at the top, inline `<style>` in the head, and one inline `<script>` (an IIFE with `'use strict'`) at the end of the body.
- `README.md`: a placeholder. Keep it accurate if you change project setup.
- `CLAUDE.md`: this file.

## Running the game

No build is needed. Open `index.html` directly in a browser, or serve the directory with any static server, for example `python3 -m http.server` from the repo root, then visit `http://localhost:8000`.

## Verifying changes

There are no automated tests, linters, or CI configured. To verify a change:

1. Load `index.html` in a browser and check the browser console for errors.
2. Play through at least one stage. Check firing, bombs, power-ups, and the boss fight if the change touches them.
3. Check the layout at phone width and at desktop width. The canvas is sized by `resize()` and the HUD and touch controls are responsive.
4. If you change difficulty, scoring, or spawn logic, test all three difficulties (`easy`, `normal`, `hard`).

## Code structure inside `index.html`

The script is organized into sections marked with `// ---------- Name ----------` comments. Keep new code in the matching section.

- **DOM references**: `const` lookups by element id at the top of the IIFE.
- **Storage keys**: `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion`. Stored in `localStorage`.
- **Adaptive performance** (`trackPerf`): averages the last 45 frame times and steps a `quality` level (0 full, 1 medium, 2 low) up or down.
- **Tuning tables**:
  - `DIFFICULTIES`: spawn rate, enemy speed and fire chance, starting lives and bombs, and boss HP multiplier.
  - `KILLS_PER_STAGE` (18): kills needed to advance a stage.
  - `THEMES`: six sky palettes, one per stage, cycled by stage number.
  - `ENEMY_TYPES`: `grunt`, `interceptor` (weaving), and `tank` (4 HP, minimum stage 2).
  - `BOSSES`: six bosses (VANGUARD, BEHEMOTH, SPECTER, DREADNOUGHT, WRAITH, TITAN), each with colors and an `hpMul`.
  - `BOSS_PATTERNS`: one array of attack functions per boss. Each function is called in sequence during the fight. Bullet helpers are `fireRadial`, `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`, `fireCross`, `fireHomingSeeded`, and `fireDoubleRing`.
- **Audio**: a Web Audio synth (`beep`, `noiseBurst`, and a `sfx` object). Audio is created lazily by `ensureAudio()`, which is called from user-gesture handlers. Keep it that way, because browsers block audio that starts without a gesture.
- **Input**: keyboard (`keys` map on `e.code`), mouse on the canvas, and a touch joystick plus FIRE and BOMB buttons.
- **Game state**: `resetGame()` initializes state, `update(dt)` advances the simulation, and `draw()` renders. `loop(now)` runs `requestAnimationFrame`, caps `dt` at 0.05 s, and calls `update` and `draw` only while `running && !paused`.
- **Power-ups**: `maybeSpawnPowerUp` creates `life`, `bomb`, or `weapon` pickups. `applyPowerUp(type)` applies them.

## Controls (keep the in-game hint in sync)

The hint text is in `#controlsHint` (around line 417). If you change a control, update that line too.

- Mouse: move to steer, click to fire.
- Arrow keys or WASD: move. Space: fire.
- X or Shift: bomb. P or Esc: pause.
- Touch: drag left to fly, tap FIRE, tap BOMB.

## Conventions

- Keep everything in `index.html` unless there is a clear reason to split it. The single-file layout is deliberate, so the game can be opened from disk.
- Match the existing style: 2-space indentation, `const` by default, arrow functions for callbacks, and short section-header comments.
- Tuning values are inline numbers in tables. Change them in the table, not in the functions that read them.
- Respect reduced motion. `flashHit` and `flashBomb` return early when `reducedMotion` is set, and `screenShake` scales its magnitude and duration down. New visual effects should check the `reducedMotion` flag the same way.
- Keep screen-reader announcements going through `announce()` and the `#srLive` region.
- Do not add external dependencies or CDN scripts without a strong reason. The game currently has none.

## Git

- Development branch for this work: `claude/gifted-tesla-yk5xr3`.
- Commit with clear messages that describe the gameplay or code change.
