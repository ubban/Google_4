# CLAUDE.md

Guidance for working in this repository.

## What this is

Sky Fighter, a Raiden-style vertical shoot-'em-up that runs in the browser. The entire game is one self-contained file, `index.html`: markup, CSS, and a single inline `<script>` that draws everything on a `<canvas>` with the 2D context. There is no build step, no package manager, no dependencies, and no test suite.

`README.md` contains only the title.

## Running it

Open `index.html` in a browser. No server is required. Nothing needs to be installed.

## Code layout (`index.html`)

- **Lines ~10–361 — `<style>`**: all CSS. Layout is a fixed full-viewport container with a centered canvas (`#gameCanvas`). HUD, overlay card, touch controls, and vignette/flash effects are absolutely positioned on top of it.
- **Lines ~362–420 — markup**: HUD stats (`#score`, `#best`, `#mult`, `#weaponName`/`#weaponLvl`, `#hearts`, `#bombs`), boss bar, combo/stage banners, top-bar buttons (mute, reduced motion, pause), touch controls, and the start overlay (`#overlay`, `#startBtn`, difficulty buttons `.diffBtn`).
- **Lines ~422–1668 — `<script>`**: one IIFE with `'use strict'`. Sections, in order:
  - DOM references and storage keys
  - Adaptive performance (`trackPerf`) — lowers `quality` when frame time is high
  - Config tables: `DIFFICULTIES`, `THEMES` (per-stage sky palettes), `ENEMY_TYPES`, `BOSSES`, `BOSS_PATTERNS`
  - Audio (`ensureAudio`, `beep`, `noiseBurst`, `sfx`), created lazily after a user gesture
  - Input (keyboard, mouse on canvas, touch joystick and buttons)
  - Game state and logic: `resetGame`, `update(dt)`, `spawnEnemy`, `spawnBoss`, `fireWeapon`, `useBomb`, `applyPowerUp`, `updateBoss`, `loseLife`, `endGame`, `togglePause`
  - Rendering: `draw()`, `drawPlane`, `drawBossShip`, `drawClouds`
  - Main loop: `loop(now)` via `requestAnimationFrame`. `dt` is clamped to 50 ms.

Game state is held in module-level `let` variables inside the IIFE (`player`, `bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`, `boss`, `bestScore`, etc.). Nothing is exported or attached to `window`.

## Conventions in this codebase

- Keep everything in `index.html` unless there is a clear reason to split it. Don't add a build system or dependencies for a change that fits in the file.
- Match the existing style: `const` by default, arrow functions for callbacks, two-space indentation, single quotes in JS, section headers written as `// ---------- Name ----------`.
- Tune gameplay through the config tables (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `THEMES`, `KILLS_PER_STAGE`) rather than hard-coding numbers in logic.
- Time-based motion multiplies by `dt` (seconds). Don't tie movement to frame count.
- Respect reduced motion: `reducedMotion` (JS flag, mirrored by the `body.reduced-motion` class) weakens screen shake (`screenShake`) and skips the hit and bomb flash effects. New flashy effects should check it too.
- Announce important state changes through `announce()` so screen readers get them via `#srLive`.

## Persistence

`localStorage` keys, all prefixed `skyfighter.`:

| Key | Holds |
| --- | --- |
| `skyfighter.bestScore` | Best score (number) |
| `skyfighter.muted` | `'1'` when muted |
| `skyfighter.difficulty` | `easy` / `normal` / `hard` |
| `skyfighter.reducedMotion` | `'1'` / `'0'`; if unset, follows `prefers-reduced-motion` |

Reads and writes are direct calls. If you add storage, keep the `skyfighter.` prefix and read through `Number(...) || default` or an explicit comparison, as the existing code does.

## Controls (for reference when changing input)

- Keyboard: arrows/WASD move, Space fires, X or Shift uses a bomb, P or Esc pauses. `Space` calls `preventDefault` to avoid page scroll.
- Mouse: move over the canvas to steer, hold to fire (`firing`).
- Touch: drag in the left zone for the virtual joystick, hold FIRE, tap BOMB. The page uses `touch-action: none` and `user-select: none`.

## Verifying changes

There are no automated tests or linters. To check a change:

1. Open `index.html` in a browser and start a game.
2. Exercise the path you changed: shooting, bombs, power-ups, stage transitions, the boss fight, pause, and game over.
3. Check the console for errors. The script runs inside an IIFE, so errors surface as uncaught exceptions.
4. For layout changes, check a narrow (phone-sized) viewport as well as desktop.

A syntax-only sanity check is possible with Node by extracting the `<script>` body, for example `node --check` on the extracted file.

## Git

- Work on the branch you were given. Commit with clear messages that describe the gameplay or UI change.
- Keep commits focused. Don't mix unrelated tuning changes with feature work.
