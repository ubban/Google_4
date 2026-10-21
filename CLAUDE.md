# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a small, dependency-free single-page HTML5 game called **Sky Fighter**, a Raiden-style vertical shoot-em-up. The entire game lives in `index.html` (~1,670 lines): markup, CSS in a `<style>` block, and one inline `<script>` wrapped in an IIFE.

Files:

- `index.html`: the whole game (markup, styles, and logic).
- `README.md`: a one-line placeholder.

There is no build step, package manager, bundler, test suite, linter, or CI configuration in this repo.

## Running the game

Open `index.html` directly in a browser. No server or install step is needed.

To verify a change, load the page and play it. The `run` skill can launch the app and drive it if you need a real browser check.

## Architecture of `index.html`

The file has three parts, in order:

1. **`<style>`** (around lines 10–360): HUD pills, the glassmorphic overlay card, touch controls, the vignette and flash effects, and the `.reduced-motion` class.
2. **`<body>` markup** (around lines 361–421): the `#container` holding the `<canvas id="gameCanvas">`, the HUD (`#hud`, `#hearts`, `#bombs`), the boss bar, the top-bar buttons (mute, reduced motion, pause), touch controls (`#touchControls`), and the start/pause `#overlay`.
3. **`<script>`** (starts around line 422): one IIFE with `'use strict'`. Nothing is exposed on `window`. Keep new code inside the IIFE.

Sections inside the script, roughly in order:

- **DOM references and constants**: `STORAGE_*` keys, adaptive `quality` tracking (`trackPerf`), `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES` (per-stage sky palettes), `ENEMY_TYPES`, `BOSSES`.
- **Audio**: `ensureAudio`, `beep`, `noiseBurst`, and the `EXPLODE_SFX` map. Audio is created lazily from a user gesture.
- **Effects and HUD helpers**: `renderHearts`, `renderBombs`, `flashHit`, `flashBomb`, `showComboPop`, `showStageBanner`, `screenShake`.
- **Input and layout**: `resize` (fits a `BASE_W` × `BASE_H` = 480 × 720 logical canvas), joystick handlers, `canvasPointFromEvent`.
- **Game state**: `resetGame`, `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`, `spawnExplosion`, `addScore`, `breakChain`.
- **Combat**: `useBomb`, `fireWeapon`, `applyPowerUp`, and the boss bullet-pattern functions (`fireRadial`, `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`, `fireCross`, `fireHomingSeeded`, `fireDoubleRing`), collected in `BOSS_PATTERNS`.
- **Simulation**: `update(dt)` advances everything. `updateBoss(dt)`, `killBoss`, `loseLife`, `endGame`, and `togglePause` handle state transitions.
- **Rendering**: `drawPlane`, `drawBossShip`, `drawClouds`, and `draw()`.
- **Main loop**: `loop(now)` calls `update` then `draw` via `requestAnimationFrame`. `dt` is clamped to 0.05 s, so long stalls slow the game instead of teleporting objects.

## Conventions

- **Style**: vanilla JavaScript (ES2015+), no modules, no frameworks, no transpilation. Use `const`/`let`, arrow callbacks for event handlers, and two-space indentation to match the file.
- **Naming**: `camelCase` for functions and variables, `SCREAMING_SNAKE_CASE` for top-level constants and tables (`DIFFICULTIES`, `ENEMY_TYPES`, `STORAGE_BEST`), and `STORAGE_` as the prefix for `localStorage` keys.
- **Data-driven tables**: add enemy types, bosses, themes, difficulties, and boss patterns as entries in the existing tables rather than adding special-case branches.
- **Comments**: section dividers use the `// ---------- Name ----------` form. Keep comments short and explain why, not what.
- **DOM access**: look elements up once in the constants block at the top of the IIFE, and reuse those references.
- **Accessibility**: announce game events to screen readers through `announce()` (the `#srLive` region). Respect the reduced-motion setting: route screen shake and flashes through `screenShake`, `flashHit`, and `flashBomb`, which scale down or skip effects when `reducedMotion` is set.

## Persistence

Player settings and the best score are stored in `localStorage` under these keys:

| Key | Meaning |
| --- | --- |
| `skyfighter.bestScore` | Highest score (number) |
| `skyfighter.muted` | `'1'` when muted |
| `skyfighter.difficulty` | `easy`, `normal`, or `hard` (default `normal`) |
| `skyfighter.reducedMotion` | `'1'` or `'0'`. If absent, the `prefers-reduced-motion` media query decides. |

Every `localStorage` access can throw in some browsers (for example, private mode or disabled storage). Wrap new reads and writes in `try`/`catch` and make the game work without them. Existing code does not do this yet, so treat it as a known gap, not a pattern to copy.

## Gameplay tuning

- Difficulty settings live in `DIFFICULTIES`: spawn timing, enemy speed, shot chance, starting lives and bombs, and boss HP multiplier.
- Stage length is `KILLS_PER_STAGE` (currently 18). Stage themes cycle through `THEMES`.
- Enemy and boss stats live in `ENEMY_TYPES` and `BOSSES`. Each boss's `hpMul` is applied on top of the difficulty's `bossHpMul`.
- Adaptive quality (`quality` 0–2) lowers visual effects when average frame time stays above about 25 ms, and raises it back when frames are fast. Any new particle or cloud effect should respect the current quality level.

## Controls (for manual testing)

- **Desktop**: mouse to steer, click to fire. Arrow keys or WASD to move. Space to fire. X or Shift for a bomb. P or Esc to pause.
- **Mobile**: drag in the left move zone to fly with the virtual joystick. Tap FIRE and BOMB.

## Making changes

- Keep the single-file structure. Do not split out modules, add a build step, or add dependencies unless the user asks for it.
- Test by playing in a browser, and cover at least: starting a game, firing and collecting power-ups, using a bomb, losing a life, reaching a boss, game over, and the best-score update. Check the touch controls on a narrow viewport. Check that the reduced-motion toggle and mute persist across reloads.
- Watch for performance regressions. Particle counts and per-frame allocations directly affect the adaptive-quality logic.
- Do not commit generated or temporary files. Keep changes scoped to what the task needs.
