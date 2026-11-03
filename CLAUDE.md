# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo is a single-page HTML5 canvas game, **Sky Fighter**: a Raiden-style vertical shoot-'em-up. Everything lives in one file:

- `index.html` (~1,670 lines): inline `<style>`, markup, and one inline `<script>` IIFE. No build step, no bundler, no dependencies.
- `README.md`: minimal project title only.

There is no package manager, test suite, linter, or CI config in the repo.

## Running the game

Open `index.html` directly in a browser, or serve the folder with any static server, for example:

```bash
python3 -m http.server 8000
```

Then visit `http://localhost:8000`.

## Code layout inside `index.html`

The script is wrapped in an IIFE with `'use strict'`. Major sections, in source order:

- **DOM references**: `const` lookups for canvas, HUD, overlay, and touch controls.
- **Storage keys**: `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion`, all via `localStorage`.
- **Adaptive performance** (`trackPerf`): tracks frame times and steps a `quality` level (0 full, 1 medium, 2 low).
- **Audio**: `ensureAudio`, `beep`, `noiseBurst`, and the `sfx` table. Audio is synthesized with WebAudio; no audio files.
- **Config tables**:
  - `DIFFICULTIES` (easy / normal / hard): spawn timing, enemy speed, shoot chance, lives, bombs, boss HP multiplier.
  - `KILLS_PER_STAGE` (18): kills needed before a stage boss spawns.
  - `THEMES`: six background palettes, cycled by stage.
  - `ENEMY_TYPES`: grunt, interceptor, tank, with `minStage` gating.
  - `BOSSES`: six boss designs, cycled by stage, with per-boss `hpMul`.
- **Input**: keyboard (`keys` map, keydown/keyup), mouse steering and fire, and touch joystick/fire/bomb buttons.
- **Game state and loop**: `resetGame`, `update(dt)`, `draw()`, `loop(now)`. `dt` is clamped to 50 ms.
- **Weapons and bullets**: `fireWeapon` (Vulcan/Laser, 3 power levels), `applyPowerUp`.
- **Bosses**: `spawnBoss`, `updateBoss`, `killBoss`, and boss attack patterns (`fireRadial`, `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`, `fireCross`, `fireHomingSeeded`, `fireDoubleRing`).
- **Flow**: `loseLife`, `endGame`, `togglePause`, `useBomb`, `breakChain` (kill-chain multiplier).
- **Rendering**: `drawPlane`, `drawBossShip`, `drawClouds`, `spawnExplosion`, screen shake and hit/bomb flashes.

## Controls (for reference when testing)

- Mouse: move to steer, click to fire.
- Arrows / WASD: move. Space: fire. X or Shift: bomb. P or Esc: pause.
- Touch: drag left side to fly, FIRE and BOMB buttons on the right.

## Conventions

- Keep it a single self-contained file unless the user asks for a restructure. Do not add a build step or external dependencies casually.
- Preserve the IIFE and `'use strict'` wrapper; new globals should not leak to `window`.
- Use the existing `DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, and `THEMES` tables for tuning rather than hard-coding numbers in logic.
- Respect `reducedMotion` (`body.reduced-motion` class) when adding new screen effects.
- Mobile matters: the layout uses `viewport-fit=cover`, safe-area insets, and `touch-action: none`. Check phone-width layout after UI changes.
- Keep the HUD accessible: announcements go through `announce()` into the `srLive` live region.

## Verification

Since there are no automated tests, verify changes by loading `index.html` in a browser and playing through at least one stage boss, a pause/resume cycle, a bomb, and a game over. For UI changes, also check a narrow (phone-sized) viewport.

## Git

- Development branch for Claude Code work: `claude/gifted-tesla-k3i6xb`.
- Commit messages should be short and descriptive; the existing history uses a one-line subject followed by bullet-point body text.
