# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Sky Fighter is a Raiden-style vertical shoot-'em-up that runs in the browser. The whole game (markup, CSS, and JavaScript) lives in a single file, `index.html`.

There is no build step, package manager, bundler, or test suite. Do not add one without asking.

## Repository layout

- `index.html` — the entire game:
  - `<style>`: HUD, overlays, touch controls, damage vignette and flash effects, and the `#container`/`#gameCanvas` layout.
  - Body markup: canvas, HUD elements (score, multiplier, weapon, hearts, bombs, boss bar), start/pause overlay, touch controls.
  - One `<script>` containing a single `'use strict'` IIFE with all game logic.
- `README.md` — placeholder, one line.

## Running and checking changes

- Open `index.html` directly in a browser, or serve the folder with any static server (e.g. `python3 -m http.server`). Nothing needs to be installed.
- There are no automated tests. Verify changes by playing the game in a browser and checking the DevTools console for errors.
- For touch behavior, use DevTools device emulation with touch enabled. Touch controls only appear when the device reports touch support.
- When touching rendering or effects, check both normal and reduced-motion modes (the Motion toggle in the HUD).

## Code structure (inside the script)

The script is organized into sections marked with `// ---------- Name ----------` banner comments. Keep new code in the matching section.

- **Adaptive performance**: `trackPerf` samples frame time over 45 frames and steps `quality` between 0 (full), 1 (medium), and 2 (low). `spawnExplosion` scales particle counts by it.
- **Config tables**: `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES`, `ENEMY_TYPES`, `BOSSES`, and `BOSS_PATTERNS`. Gameplay tuning belongs here rather than inline in logic.
- **Audio**: `ensureAudio`, `beep`, `noiseBurst`, and the `sfx` object. All sounds are synthesized with the Web Audio API, so there are no audio asset files. Audio is created lazily on the first user gesture.
- **Input**: keyboard (arrows/WASD to move, Space to fire, X/Shift for bomb, P/Escape to pause), a touch joystick, and mouse steering. Keyboard and joystick input take priority over mouse steering.
- **Game state**: `resetGame` and the module-level state variables (`player`, `bullets`, `enemies`, `enemyBullets`, `powerUps`, `boss`, `score`, `lives`, `stage`, and so on).
- **Update and draw**: `update(dt)` runs the simulation and `draw()` renders the frame. The `loop` function caps `dt` at 0.05 s and skips update and draw while paused or stopped.
- **Boss**: `spawnBoss`, `updateBoss`, and `killBoss`. Attack patterns come from `BOSS_PATTERNS`, indexed by `(stage - 1) % 6`.

## Game rules

Keep these in mind when changing gameplay. Update this list if you change any of them.

- **Logical canvas**: 480×720. `resize()` scales it with CSS, capped at 1.6×. All game coordinates use the logical size.
- **Difficulty** (`easy`, `normal`, `hard`) sets lives, starting bombs, spawn rate, enemy speed, enemy fire chance, and boss HP multiplier.
- **Stages**: a stage ends after `KILLS_PER_STAGE` (18) kills. A boss warning appears, and the boss spawns 1.2 s later. Boss HP is `(70 + stage × 35)` × difficulty multiplier × boss design multiplier. Defeating a boss advances the stage and cycles the sky theme through `THEMES`.
- **Enemies**: `grunt` from stage 1; `interceptor` (weaving) from stage 1; `tank` (4 HP) from stage 2.
- **Power-ups** drop only from destroyed enemies. Roll outcomes: life (6%), bomb (8%), weapon (20%, cumulative with the others).
- **Weapons**: the weapon pickup raises the level from 1 to 3. At level 3 it toggles between Vulcan and Laser, and the level resets to 1.
- **Scoring**: points are multiplied by `1 + min(9, floor(chain / 5))`. The chain resets when the player is hit or when `chainTimer` (2.2 s, refreshed by each kill) runs out.
- **Getting hit**: costs one life, grants 1.5 s of invulnerability, and breaks the chain.
- **Bombs** clear all enemies and enemy bullets and deal 24 damage to the boss.
- **Persistence**: the only saved state is in `localStorage`, under these keys:
  - `skyfighter.bestScore`
  - `skyfighter.muted`
  - `skyfighter.difficulty`
  - `skyfighter.reducedMotion`

  Keep these keys stable, because changing them would reset players' saved data.

## Conventions

- Match the existing style: 2-space indentation, single quotes, `const` for config tables, and short section banner comments.
- Keep everything in `index.html` unless the user asks for a split.
- Preserve the `'use strict'` IIFE. Avoid leaking globals.
- Keep DOM lookups at the top of the script, next to the existing `getElementById` calls.
- Keep the layout working at phone width with no horizontal scroll. The page uses `overflow: hidden` and `touch-action: none`, so new UI must fit within the viewport.
- Do not introduce external scripts, fonts, or network calls. The game is self-contained.
