# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Google_4 is a small static web project. Its only game is **Sky Fighter**, a
Raiden-style vertical shoot-em-up in a single file, `index.html` (~58 KB). There is
no build step, no package manager, no bundler, and no test suite.

Files:

- `index.html` - the entire app: inline `<style>`, markup, and one inline `<script>`.
- `README.md` - a one-line title only.
- `CLAUDE.md` - this file.

## Running it

Open `index.html` directly in a browser, or serve the directory with any static server
(for example `python3 -m http.server`). Nothing needs to be installed.

There are no automated tests or linters configured. Verify changes by loading the page
and playing it, ideally with both mouse/keyboard and a touch-sized viewport.

## Architecture of `index.html`

The game script is one IIFE (`'use strict'`) inside the `<script>` tag at the bottom of
the file. Rough layout, in source order:

1. **DOM references** - `const xxxEl = document.getElementById(...)` for the HUD, overlay,
   touch controls, and boss bar. IDs in the markup must stay in sync with these lookups.
2. **Persistence keys** - `STORAGE_BEST`, `STORAGE_MUTE`, `STORAGE_DIFF`,
   `STORAGE_REDUCED` (all prefixed `skyfighter.`) in `localStorage`. Changing a key
   resets that user's saved value.
3. **Config tables** - `DIFFICULTIES` (spawn rates, enemy speed/fire chance, lives,
   bombs, boss HP multiplier), `KILLS_PER_STAGE`, `THEMES` (sky palettes per stage),
   `ENEMY_TYPES` (grunt / interceptor / tank), and `BOSSES` (six bosses, cycled by stage).
4. **Audio** - `ensureAudio`, `beep`, `noiseBurst`, and an `EXPLODE_SFX` map. Audio is
   created lazily on the first user gesture.
5. **UI helpers** - hearts/bomb rendering, hit flash, combo pop, stage banner, screen
   shake, mute and reduced-motion toggles, difficulty selection, and `announce` for the
   screen-reader live region.
6. **Input** - keyboard (arrows/WASD, Space fire, X/Shift bomb, P/Esc pause), mouse and
   pointer steering, and a touch joystick plus FIRE/BOMB buttons.
7. **Simulation** - `resetGame`, `update(dt)` (movement, collisions, scoring, chain
   multiplier, power-ups), `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`, `useBomb`,
   `fireWeapon` (Vulcan/Laser, levels 1-3), `applyPowerUp`, `loseLife`, `endGame`,
   `togglePause`.
8. **Boss patterns** - `fireRadial`, `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`,
   `fireCross`, `fireHomingSeeded`, `fireDoubleRing`, collected in `BOSS_PATTERNS` and
   driven by `updateBoss` / `killBoss`.
9. **Rendering** - `drawPlane`, `drawBossShip`, `drawClouds` (parallax layers), and
   `draw()`. A `quality` level is adjusted from measured frame time by `trackPerf`.
10. **Main loop** - `loop(now)` clamps `dt` to 50 ms, updates only when `running && !paused`,
    then draws. Logical resolution is `BASE_W x BASE_H` (480 x 720); `resize()` scales the
    canvas to fit the window.

Game state is held in module-level `let` variables inside the IIFE. There are no
classes, modules, or external dependencies.

## Conventions

- Keep everything in `index.html` unless the user asks for a split. Do not add a build
  step or npm dependencies for this project without asking.
- Match the existing style: two-space indentation, single quotes in JS, `const` by default,
  and small named functions grouped by the sections above.
- Tuning values live in the config tables (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`,
  `THEMES`). Prefer changing those over hard-coding numbers in logic.
- Respect the accessibility hooks: `prefers-reduced-motion`, the `reduced-motion` body class,
  the `srLive` live region, and the `sr-only` text. New visual effects like shake or flash
  should be gated by `reducedMotion`.
- Keep the mobile layout working: the touch controls appear on coarse-pointer devices, and
  the HUD must not overlap the controls at phone width.

## Git

- Development branch for this work: `claude/gifted-tesla-myvhte`. Push there, not to
  other branches, unless the user says otherwise.
- Commit messages in this repo are imperative, descriptive subject lines followed by a
  bulleted body, as in the existing history.
