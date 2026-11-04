# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up that runs in the browser. The whole game lives in one file, `index.html`: inline CSS in `<style>`, markup in `<body>`, and all game logic in a single IIFE inside one `<script>` block. It draws on a `<canvas id="gameCanvas">` using the 2D context.

Files:

- `index.html` — the entire game (about 1,670 lines).
- `README.md` — minimal project title.
- `CLAUDE.md` — this file.

There is no build step, package manager, bundler, linter, or test suite. Do not add one unless asked.

## Running and verifying

- Open `index.html` directly in a browser, or serve the directory with any static server (for example `python3 -m http.server`).
- Verify changes by playing them. Check the title and overlay, a full stage, the boss fight, pause (P / Esc), bomb (X / Shift), game over, and the best-score persistence.
- For touch behavior, test a touch device or emulate touch in devtools. Touch controls appear only when `'ontouchstart' in window` or `navigator.maxTouchPoints > 0`.

## Code layout inside `index.html`

The script is organized into commented sections. Keep new code in the matching section:

- **Setup / DOM refs** — cached elements and `STORAGE_*` keys at the top of the IIFE.
- **Audio** — `ensureAudio`, `beep`, `noiseBurst`, and the SFX table (`EXPLODE_SFX`). Audio is created lazily on the first user gesture.
- **Config tables** — `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES` (sky palettes per stage), `ENEMY_TYPES`, and `BOSSES`. Tune balance here, not in logic.
- **HUD helpers** — `renderHearts`, `renderBombs`, `flashHit`, `showComboPop`, `showStageBanner`, `screenShake`.
- **Layout** — `BASE_W = 480`, `BASE_H = 720`, scaled to fit the viewport in `resize()`.
- **Input** — keyboard (`keys` map keyed by `e.code`; P/Esc pause, X/Shift bomb), mouse on the canvas (`mousemove` sets the target via `canvasPointFromEvent`; mousedown fires), and touch controls (joystick, fire, bomb buttons).
- **Game state** — `resetGame()` initializes the state. The main mutable state is declared near line 865.
- **Entities and spawning** — `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`, `spawnExplosion`.
- **Combat and scoring** — `addScore`, `breakChain` (combo/kill-chain multiplier), `useBomb`, `fireWeapon`.
- **Update / draw** — `update(dt)` advances the simulation; `draw()` renders. `loop(now)` runs `requestAnimationFrame`, clamps `dt` to 0.05 s, and calls `update` then `draw` only when running and not paused.
- **Bosses** — `BOSS_PATTERNS` holds the bullet patterns (radial, aimed spread, columns, spiral, cross, homing, double ring). `updateBoss` and `killBoss` manage the fight.
- **Lifecycle** — `loseLife`, `endGame`, `togglePause`.

Rendering helpers `drawPlane`, `drawBossShip`, and `drawClouds` are self-contained; keep visual tweaks there.

## Gameplay rules worth knowing

- Every `KILLS_PER_STAGE` (18) kills within a stage triggers that stage's boss (spawned about 1.2 s later). Defeating it advances to the next stage. The stage theme comes from `THEMES`, cycled with `(stage - 1) % THEMES.length`.
- Enemies: `grunt` and `interceptor` appear from stage 1; `tank` (4 HP) from stage 2.
- Difficulty (`easy` / `normal` / `hard`) sets spawn rate, enemy shoot chance, starting lives and bombs, and boss HP multiplier.
- Weapons have 3 power levels (Vulcan / Laser). Power-ups include weapon level, bomb, and extra life.
- Bombs clear the screen of bullets and enemies and damage bosses.

## Persistence

`localStorage` keys (all prefixed `skyfighter.`):

- `bestScore` — high score, written in `endGame`.
- `muted` — `'1'` or `'0'`.
- `difficulty` — last selected difficulty.
- `reducedMotion` — `'1'` or `'0'`. If unset, defaults to the `prefers-reduced-motion` media query.

Reads and writes to `localStorage` are not wrapped in try/catch. If you touch them, consider that private or blocked storage can throw.

## Accessibility and UX conventions

- Screen-reader announcements go through `announce()`, which writes to the `#srLive` live region.
- The `reduced-motion` body class disables or softens shake, pulses, and entrance animations. Respect it in new animations.
- Keep the HUD in the pill/glass style already established. Stage banners, combo pops, and the vignette are all CSS-driven and toggled by class.

## Working guidelines

- Make minimal, targeted edits. Do not reformat unrelated code or reorder sections.
- Keep everything in `index.html` unless there is a clear reason to split it. Avoid adding external dependencies or CDN scripts.
- When tuning balance, change the config tables (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `KILLS_PER_STAGE`) instead of hard-coding numbers in logic.
- Preserve the `'use strict'` IIFE wrapper and the existing naming style (camelCase functions, UPPER_CASE config constants, `STORAGE_*` keys).
- After any gameplay or rendering change, play through at least one stage and a boss before considering the change done.
