# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo contains **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs in the browser. The entire game is a single self-contained file, `index.html` (inline `<style>` and one inline `<script>`, about 1,670 lines). There is no build step, no bundler, no package manager, no dependencies, and no test suite.

Other files:

- `README.md` is a placeholder. It currently contains only a title and a timestamp line.

## Running the game

Open `index.html` directly in a browser, or serve the directory with any static server. For example:

```
python3 -m http.server 8000
```

Then visit `http://localhost:8000/index.html`.

## Verifying changes

There are no automated tests or linters configured. To verify a change:

1. Load `index.html` in a browser and check the browser console for errors.
2. Play through at least one start, one stage transition, and one game over.
3. For layout changes, check both a desktop-sized and a phone-sized viewport. Touch controls are only shown on touch devices.

## Code structure (`index.html`)

- **Markup** (`<body>`): a `#container` holds the `<canvas id="gameCanvas">`, the HUD (`#hud`, `#bossBarWrap`, `#comboPop`, `#stageBanner`), the top-right buttons (`#muteBtn`, `#motionBtn`, `#pauseBtn`), touch controls (`#touchControls`), and the start/pause overlay (`#overlay`).
- **Script**: everything lives in one IIFE with `'use strict'`. Nothing is exported to `window`.
  - **Config tables** near the top: `DIFFICULTIES` (spawn rate, enemy speed, shot chance, lives, bombs, boss HP multiplier), `KILLS_PER_STAGE` (18), `THEMES` (six background palettes, cycled by stage), `ENEMY_TYPES` (grunt, interceptor, tank), and `BOSSES` (six boss designs, cycled by stage).
  - **Audio**: a Web Audio synth (`ensureAudio`, `beep`, `noiseBurst`, and the `sfx` object). No audio files are loaded.
  - **Rendering**: `draw()` and the helpers `drawPlane`, `drawBossShip`, and `drawClouds`. Drawing uses the 2D canvas context on a fixed 480×720 logical resolution (`BASE_W`, `BASE_H`), scaled by `resize()`.
  - **Simulation**: `update(dt)` handles movement, firing, collisions, enemy and bullet updates, and power-up pickup. `updateBoss(dt)` handles boss phases. Boss attack patterns are the `BOSS_PATTERNS` array, which uses helpers like `fireRadial`, `fireAimedSpread`, and `fireSpiralBurst`.
  - **Game state**: plain module-level `let` variables (`score`, `lives`, `stage`, `kills`, `player`, `enemies`, `bullets`, `enemyBullets`, `powerUps`, `boss`, and so on). `resetGame()` reinitializes them.
  - **Loop**: `loop(now)` is driven by `requestAnimationFrame`. `dt` is clamped to 50 ms. `trackPerf` adjusts a `quality` level (0 to 2) when frame times drift, which the drawing code uses to reduce effects.
  - **Input**: keyboard (`keys` map plus `keydown`/`keyup`), mouse on the canvas, and a touch joystick plus FIRE and BOMB buttons.

## Game rules worth knowing

- **Weapons**: two types, `vulcan` (spread shot) and `laser` (piercing). Each has levels 1 to 3. A weapon power-up raises the level. At level 3, the next weapon power-up switches to the other weapon type and resets the level to 1.
- **Power-ups**: each destroyed enemy may drop one. `life` is a 6% chance, `bomb` 8%, and `weapon` 20%. Rates are set in `maybeSpawnPowerUp`.
- **Bombs**: clear all enemy bullets, destroy every on-screen enemy (each adds 5 points), and deal 24 damage to the boss. Triggered by X, Shift, or the BOMB button. Bombs are limited by `maxBombs`, which comes from the difficulty setting.
- **Stages and bosses**: after `KILLS_PER_STAGE` kills, the boss is warned and spawns about 1.2 s later. Defeating the boss increments `stage`, adds 500 points, and changes the theme.
- **Chains and multiplier**: each kill extends a chain (`chainTimer` = 2.2 s). The multiplier is `1 + min(9, floor(chain / 5))`. The chain resets through `breakChain()`, which runs when the timer expires and on other events such as losing a life.
- **Pause**: P or Esc, or the pause button.

## Persistence

`localStorage` keys (all prefixed `skyfighter.`):

- `bestScore`: the high score
- `muted`: `'1'` when muted
- `difficulty`: `easy`, `normal`, or `hard`
- `reducedMotion`: stores the reduced-motion toggle. When unset, the `prefers-reduced-motion` media query decides the default.

All reads and writes go directly through `localStorage`. If you add new keys, use the `skyfighter.` prefix and keep the key names as constants next to the existing `STORAGE_*` constants.

## Conventions

- Keep the game in a single file unless the user asks for a restructure. Do not add a build step or dependencies without asking.
- Match the existing style: 2-space indentation, `const` by default, arrow functions for callbacks, and short `// ----------` section banners.
- Add new enemy, boss, or theme entries to the existing config tables rather than hard-coding them in logic.
- Keep the game playable with keyboard, mouse, and touch. Any new control needs all three paths or a clear reason it does not.
- Preserve accessibility hooks: the `#srLive` aria-live region receives announcements through `announce()`, and the reduced-motion toggle should gate screen flashes and reduce screen shake (see `screenShake`).
- Do not introduce external network requests. The game is fully offline.

## Git

- Development branch for this work: `claude/gifted-tesla-5o2y0m`. Commit and push there.
- Use clear, descriptive commit messages.
