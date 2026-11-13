# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo is a small, dependency-free browser game, **Sky Fighter**: a Raiden-style vertical shoot-'em-up. The entire game lives in one file, `index.html` (~1,670 lines): inline CSS in `<style>`, markup, and one inline `<script>` IIFE. There is no build step, no package manager, no test suite, and no framework.

Files:

- `index.html` — the whole game (HTML, CSS, JS).
- `README.md` — minimal placeholder.
- `CLAUDE.md` — this file.

## Running it

Open `index.html` directly in a browser, or serve the directory with any static server (e.g. `python3 -m http.server`). Nothing needs to be installed.

## Code layout inside `index.html`

- **DOM refs (top of script):** all elements are looked up once by ID (`gameCanvas`, `overlay`, `hearts`, `bombs`, `touchControls`, `bossBarWrap`, etc.). If you add UI, add the element in markup and grab it here.
- **Storage keys:** `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion`. Changing a key's name resets that player's saved value — don't rename them casually.
- **Adaptive quality:** `trackPerf` samples frame times and steps a `quality` level (0–2) up or down to shed effects on slow devices.
- **Tuning data (constants, top of script):**
  - `DIFFICULTIES` — spawn rates, enemy speed and fire chance, lives, bombs, boss HP multiplier for `easy` / `normal` / `hard`.
  - `KILLS_PER_STAGE` — kills needed to advance a stage.
  - `THEMES` — six sky palettes, cycled per stage.
  - `ENEMY_TYPES` — `grunt`, `interceptor` (weaving), `tank` (4 HP, unlocks at stage 2).
  - `BOSSES` — six bosses, each with colors and an `hpMul`.
  - `BOSS_PATTERNS` — bullet patterns (radial, aimed spread, columns, spiral, cross, homing, double ring) that bosses cycle through.
- **Audio:** Web Audio synthesis only (`beep`, `noiseBurst`, the `sfx` table). No audio files. `ensureAudio()` must run from a user gesture before sound plays.
- **Game state and logic:** `resetGame`, `update(dt)` (movement, collisions, spawning, scoring), `applyPowerUp`, `useBomb`, `fireWeapon`, `updateBoss`, `killBoss`, `loseLife`, `endGame`, `togglePause`.
- **Rendering:** `draw()` on a 2D canvas, plus helpers `drawPlane`, `drawBossShip`, `drawClouds`. The canvas is a fixed logical size `BASE_W × BASE_H` (480×720), scaled by `resize()`.
- **Loop:** `loop(now)` runs via `requestAnimationFrame`, clamps `dt` to 50 ms, and only updates/draws while `running && !paused`.
- **Input:** keyboard (`keys[e.code]`), mouse on the canvas, and touch — a virtual joystick (`moveZone`), a fire button, and a bomb button. Touch and mouse paths are separate; keep both working when changing controls.
- **Accessibility:** `announce()` writes to the `srLive` live region. Reduced motion defaults to the OS `prefers-reduced-motion` setting and can be toggled in-game. When on, it adds the `reduced-motion` body class, skips the hit/bomb screen flashes (`flashHit`, `flashBomb`), and scales screen shake down (`screenShake`). Keep new visual effects behind the same check.

## Conventions

- Keep everything in `index.html` unless there is a clear reason to split. Do not add a build tool, bundler, or npm dependency without asking.
- Preserve the single IIFE with `'use strict'`. Module-level state is `let` variables inside it; avoid new globals on `window`.
- Match the existing style: 2-space indent, single quotes in JS, `const` by default, `let` for mutable game state, arrow functions for callbacks.
- Put tunable numbers in the constants blocks above rather than burying them in logic.
- Gameplay values are balance-sensitive. When changing spawn rates, HP, or scoring, say so explicitly in the commit or PR so it can be reviewed as a balance change.
- Use `localStorage` only through the existing `STORAGE_*` keys, and keep reads/writes tolerant of missing values (the current `|| default` pattern).

## Testing and verification

There are no automated tests. Verify changes by:

1. Opening the page in a browser (or serving it locally) and checking the browser console for errors.
2. Playing through start → stage clear → boss → game over, on each difficulty if the change touches balance.
3. Checking both keyboard and touch controls if input code changed.
4. Checking mobile widths (the layout uses `viewport-fit=cover` and a fixed logical canvas) if UI or CSS changed.

If you add a test harness, document how to run it here.

## Git workflow

- Default branch is the one the repo was cloned from. Develop on the branch assigned for the session (currently `claude/gifted-tesla-puehe8`) and push there with `git push -u origin <branch>`.
- Write short, imperative commit messages that describe the gameplay or code effect (e.g. "Slow tank bullets on hard difficulty", "Fix pause overlay on resize").
- Do not open pull requests unless explicitly asked.
