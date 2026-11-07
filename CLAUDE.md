# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo is **Sky Fighter**, a single-file, browser-based vertical shoot-'em-up (Raiden-style). There is no build step, no package manager, and no test suite.

- `index.html` — the entire game: inline `<style>`, markup, and one inline `<script>` (an IIFE with `'use strict'`).
- `README.md` — minimal project title.

To run it, open `index.html` in a browser. No server is required.

## Repository layout

| Path | Purpose |
| --- | --- |
| `index.html` | Full game (markup, CSS, JS). About 1,670 lines. |
| `README.md` | Project title only. |
| `CLAUDE.md` | This file. |

Git: the default branch is `main`. Feature work goes on the branch named in the session instructions (currently `claude/gifted-tesla-64gjzh`).

## Architecture of `index.html`

Read the file top to bottom in this order when orienting yourself:

1. **Markup (approx. lines 361–421):** `#container` holds `#gameCanvas`, the HUD (`#hud`, with hearts, bombs, weapon, multiplier, score, and best), the boss bar (`#bossBarWrap`), the combo and stage banners, touch controls (`#touchControls`: joystick, fire, bomb), and the start/pause overlay (`#overlay`).
2. **Constants (approx. lines 485–516):**
   - `DIFFICULTIES` — `easy` / `normal` / `hard`, controlling spawn rate, enemy speed, enemy fire chance, starting lives and bombs, and boss HP multiplier.
   - `KILLS_PER_STAGE` (18) — kills needed before a boss spawns.
   - `THEMES` — six background palettes, cycled by stage.
   - `ENEMY_TYPES` — `grunt`, `interceptor`, `tank`, each with `minStage` gating.
   - `BOSSES` — six boss designs, cycled by stage. Each has an `hpMul`.
   - `BOSS_PATTERNS` (approx. line 1372) — bullet patterns per boss design.
   - Canvas base size is `BASE_W = 480`, `BASE_H = 720`, scaled to fit the viewport.
3. **Audio (approx. lines 545–680):** Web Audio helpers (`ensureAudio`, `beep`, `noiseBurst`, and the `sfx` object). Audio is created lazily on the first user gesture.
4. **Game state and lifecycle:** `resetGame` (approx. line 875), `update(dt)` (approx. line 1148), `updateBoss`, `killBoss`, `loseLife`, `endGame`, and `togglePause`.
5. **Spawning and combat:** `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`, `applyPowerUp`, `fireWeapon`, `useBomb`, and the `fire*` boss-pattern helpers.
6. **Rendering:** `draw()` (approx. line 1520), with `drawPlane`, `drawBossShip`, `drawClouds`, and `resize()`.
7. **Main loop:** `loop(now)` (approx. line 1640) drives `update` and `draw` with a delta time. `trackPerf` monitors frame timing.
8. **Input:** keyboard (`keydown` and `keyup` on `window`), mouse on the canvas, and touch joystick and buttons. Keyboard: arrows or WASD to move, Space to fire, `X` or Shift for a bomb, `P` or Escape to pause.

### Persistence

State is stored in `localStorage` under the `skyfighter.*` keys:

- `skyfighter.bestScore`
- `skyfighter.muted`
- `skyfighter.difficulty`
- `skyfighter.reducedMotion`

Accessing `localStorage` is not wrapped in `try/catch`. If you add new storage, consider whether private-mode or blocked-storage browsers need protection.

### Accessibility

- A `#srLive` ARIA live region receives announcements through `announce()`. Use it for significant game events such as stage changes, boss defeat, and game over.
- A reduced-motion toggle (`motionBtn`) and a mute toggle (`muteBtn`) exist. Respect `reducedMotion` when adding new screen shake, flashes, or animations.

## Conventions

- **Single file.** Keep the game in `index.html` unless the user asks for a split. Do not add a bundler or framework.
- **No external dependencies.** The page uses no CDN scripts. Keep it that way unless asked.
- **Style:** two-space indentation, single quotes in JS, `const` by default, and arrow functions for callbacks. Match the surrounding code.
- **Tuning values** belong in the `DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, and `THEMES` tables rather than as magic numbers in logic.
- **Touch and desktop parity.** Any new control needs both a keyboard path and a touch or pointer path.
- **Mobile layout.** The page is `overflow: hidden` and `touch-action: none`. Verify changes at phone width. Do not introduce page scrolling.

## Verifying changes

There is no automated test or lint setup. To verify:

1. Open `index.html` in a browser, or serve the directory with any static server.
2. Play through at least one stage and one boss on each difficulty if the change touches balance.
3. Check the console for errors. Pause, resume, mute, change difficulty, and reach game over to exercise the state transitions.
4. Check layout in a narrow viewport (around 375px wide) and in landscape.

If you cannot run a browser, say so explicitly in your summary. Do not claim the game was tested when it was only read.

## Git and workflow

- Commit with clear, descriptive messages.
- Push with `git push -u origin <branch>`.
- Do not open a pull request unless the user asks for one.
- Do not commit scratch files. Temporary output goes in the session scratchpad, not the repo.
