# CLAUDE.md

Guidance for working in this repository.

## Project overview

Sky Fighter is a Raiden-style vertical shoot-em-up built as a **single self-contained HTML file**: `index.html`. It is a 2D HTML5 canvas game with no framework, no bundler, no npm dependencies, and no build step. `README.md` is a placeholder.

There are no tests, linters, or CI configured. Verify changes by opening `index.html` in a browser and playing.

## File layout (`index.html`)

- **Lines ~1–360**: `<style>` block. Pill-style HUD, glassmorphic overlay card, touch controls, and responsive layout.
- **Lines ~362–420**: DOM markup. `#container` holds the canvas, the HUD (`#hud`), the boss bar, the stage banner, touch controls (`#touchControls`), and the start/pause overlay (`#overlay`).
- **Lines ~422–1669**: one `<script>` containing a single IIFE (`(function () { 'use strict'; ... })();`). Nothing is exported or attached to `window`.

Inside the IIFE, the code is grouped into these sections:

| Section | Purpose |
|---|---|
| Config constants | `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES`, `ENEMY_TYPES`, `BOSSES`, `BOSS_PATTERNS` |
| Adaptive perf | `trackPerf` adjusts `quality` (0 full, 1 medium, 2 low) from average frame time |
| Audio | `ensureAudio`, `beep`, `noiseBurst`, and the `sfx` table. Sounds are synthesized, not loaded from files. |
| Input | keyboard (`keys`), mouse, joystick on touch (`joystickStart/Move/End`), and touch buttons |
| Game state | `resetGame`, `update(dt)`, `draw()`, `loop(now)` |
| Spawning and combat | `spawnEnemy`, `spawnBoss`, `fireWeapon`, `applyPowerUp`, `useBomb`, `breakChain` |
| Boss patterns | `fireRadial`, `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`, `fireCross`, `fireHomingSeeded`, `fireDoubleRing` |

## Core mechanics

- **Difficulty** (`DIFFICULTIES`): `easy`, `normal`, `hard`. Each sets spawn rate, enemy speed and fire chance, starting lives, starting bombs, and a boss HP multiplier.
- **Stages**: after `KILLS_PER_STAGE` (18) stage kills, a boss spawns. Defeating it increments `stage`. `THEMES` and `BOSSES` are both indexed by `(stage - 1) % length`, so they cycle.
- **Bosses**: `spawnBoss()` takes its design from `BOSSES`, with HP = `(70 + stage × 35) × difficulty.bossHpMul × design.hpMul`. Each design's attack patterns come from `BOSS_PATTERNS[designIndex]`.
- **Enemies**: `grunt`, `interceptor` (weaves), and `tank` (4 HP). Each has a `minStage`.
- **Weapons**: Vulcan and Laser, with three power levels (`player.level`).
- **Bombs**: limited per difficulty. A bomb clears the screen and damages bosses. Triggered by X or Shift, or the BOMB button.
- **Kill-chain multiplier**: `addScore` and `breakChain` control the score multiplier shown in the HUD.
- **Power-ups**: weapon level, bomb, and extra life (capped at `maxLives`). Handled in `applyPowerUp`.
- **Pause**: P or Esc, or the pause button.

## Persistence

All persistence uses `localStorage`. Keys are namespaced with `skyfighter.`:

- `skyfighter.bestScore`
- `skyfighter.muted`
- `skyfighter.difficulty`
- `skyfighter.reducedMotion`

Reads and writes are not wrapped in try/catch. If you add storage calls, wrap them, because storage can be unavailable in private windows or when site data is blocked.

## Rendering and sizing

- The logical canvas is fixed at `BASE_W × BASE_H` = **480 × 720**. `resize()` scales the CSS size to fit the window, capped at 1.6×. All game coordinates use this logical size.
- Pointer and touch input is converted to canvas coordinates with `canvasPointFromEvent`. Don't use raw client coordinates for gameplay.
- The timestep is clamped with `dt = Math.min(dt, 0.05)`. Keep game logic frame-rate independent by using `dt`.
- Screen shake (`screenShake`) and reduced motion are honored. When `reducedMotion` is on, the `reduced-motion` body class is set. Keep new effects compatible with it.

## Conventions

- Keep everything in `index.html` unless there is a clear reason to split. The single-file design is intentional.
- Use `const`/`let`, `'use strict'`, and the existing IIFE scope. Don't add globals.
- Match the existing style: short section headers like `// ---------- Name ----------`, compact one-line config objects, and `camelCase` functions.
- Add new tunable numbers to the config objects near the top of the script rather than hard-coding them in logic.
- Gameplay state resets in `resetGame()`. Add any new state there.
- Keep accessibility hooks working. Announcements go through `announce()` into the `#srLive` live region.

## Working on this repo

- Branch for ongoing work: `claude/gifted-tesla-c7olzc`.
- There is no build command. To check syntax, run `node --check` on the extracted script, or open the page in a browser.
- Do not add dependencies or a build system without a clear request.
