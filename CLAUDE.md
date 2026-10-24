# CLAUDE.md

Guidance for working in this repository.

## Project overview

This repo contains **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs in the browser. The entire game (HTML, CSS, and JavaScript) lives in a single file, `index.html`. `README.md` is a placeholder.

There is no build step, package manager, bundler, linter, or test suite. Do not add one unless asked.

## Running and verifying

- Open `index.html` directly in a browser, or serve the directory with any static server (for example `python3 -m http.server`).
- There are no automated tests. Verify behavior by playing the game in a browser. Check at least:
  - Start, pause (`P` / `Esc`), and game over flows
  - Weapon and bomb power-ups, and the stage-clear and boss sequence
  - Touch controls, if you changed input or layout code
  - The mute, reduced-motion, and difficulty toggles
- Keep the game playable at phone width as well as desktop width.

## File layout (`index.html`)

The file has three sections, in order:

1. **`<style>`** (lines 10–359): all CSS, including the HUD, overlay, touch controls, and the `reduced-motion` and `critical` states.
2. **`<body>` markup** (lines 361–421): `#container` holds the `<canvas id="gameCanvas">`, the HUD (`#hud`), boss bar, banners, top-bar buttons, touch controls, and the `#overlay` start/pause/game-over card.
3. **`<script>`** (lines 422–1667): one IIFE wrapping all game logic, with `'use strict'`.

Within the script, code is grouped roughly as follows:

| Area | Key identifiers |
|---|---|
| DOM refs and constants | `STORAGE_*` keys, `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES`, `ENEMY_TYPES`, `BOSSES` |
| Adaptive quality | `quality`, `trackPerf` (lowers effects quality when frame time is high) |
| Audio | `ensureAudio`, `beep`, `noiseBurst`, `sfx` (Web Audio, synthesized, no asset files) |
| HUD and feedback | `renderHearts`, `renderBombs`, `showComboPop`, `showStageBanner`, `screenShake`, `flashHit`, `announce` (screen-reader live region) |
| Input | `keys` (keyboard via `e.code`), mouse on canvas, `joystickStart/Move/End`, touch buttons |
| Game state | `resetGame`, `score`, `lives`, `bombCount`, `stage`, `stageKills`, `bossWarned`, `player` |
| Spawning | `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`, `spawnExplosion` |
| Combat | `fireWeapon`, `useBomb`, `addScore`, `breakChain`, `loseLife`, `killBoss` |
| Boss patterns | `BOSS_PATTERNS` and the `fire*` helpers (radial, aimed spread, columns, spiral, cross, homing, double ring) |
| Update and draw | `update(dt)`, `updateBoss(dt)`, `draw()`, `drawPlane`, `drawBossShip`, `drawClouds` |
| Lifecycle | `endGame`, `togglePause`, `loop(now)` (requestAnimationFrame driver) |

### Game rules that are easy to break

- **Stages:** a stage ends after `KILLS_PER_STAGE` (18) kills. A boss then spawns 1.2 s later (`bossWarned` prevents a second spawn). Defeating the boss advances `stage` and cycles the background `THEMES`.
- **Difficulty** (`DIFFICULTIES`): tunes spawn timing, enemy speed, enemy fire chance, starting lives and bombs, and boss HP multiplier. The selected level persists in `localStorage`.
- **Power-ups** (`applyPowerUp`): `life` (capped at `maxLives`), `bomb` (capped at `maxBombs`), and `weapon`. Weapon upgrades go level 1 → 2 → 3, then switch between `vulcan` and `laser` and reset to level 1.
- **Damage:** `loseLife` grants 1.5 s of invulnerability and breaks the kill combo chain.
- **Persistence:** only best score, mute, difficulty, and reduced-motion preference are saved, under `skyfighter.*` keys in `localStorage`. Every access should tolerate storage being unavailable.
- **`dt` is clamped** to 0.05 s in `loop`. Game speeds are per-second values multiplied by `dt`, so keep new movement code frame-rate independent.

## Conventions

- Match the existing style: 2-space indentation, single quotes in JS, `const`/`let` (no `var`), arrow functions for callbacks, and short section comments in the form `// ---------- Name ----------`.
- Keep everything in `index.html` unless the user asks for a split. Do not introduce frameworks, npm dependencies, or external asset files. Audio is synthesized and graphics are drawn on canvas.
- Preserve accessibility hooks: announce major state changes through `announce()`, keep `aria-*` attributes and the `role="status"` live region, and respect `reducedMotion` for shake and flash effects.
- Any new UI text or control needs a `title` or label, and must work with both keyboard and touch input.
- When adding a new enemy, boss, or power-up, extend the existing table (`ENEMY_TYPES`, `BOSSES`, `BOSS_PATTERNS`, or the `applyPowerUp` branches) rather than adding a parallel code path.

## Git

- Development branch for this session: `claude/gifted-tesla-qgpm7f`. Push there with `git push -u origin <branch>`.
- Write clear commit messages that describe the change and its reason.
- Do not open pull requests unless the user asks.
