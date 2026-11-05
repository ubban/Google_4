# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo is a single-page HTML5 Canvas game, **Sky Fighter**: a Raiden-style vertical shoot-'em-up.

- The entire app lives in `index.html` (~1,670 lines): inline `<style>`, the page markup, and one inline `<script>` IIFE.
- There is no build step, package manager, test suite, or framework. Nothing to install.
- `README.md` is a placeholder (one line). Keep it accurate if you change how the project is run or shipped.

## Running and verifying

- Open `index.html` directly in a browser, or serve the directory with any static server (e.g. `python3 -m http.server`).
- Verify gameplay changes in a real browser, not just by reading the code. Check the title overlay, Start, keyboard controls (arrows/WASD to move, Space to fire, X or Shift to bomb, P or Escape to pause), game over, and the best-score update.
- Test mobile layout changes at phone width (touch controls: joystick on the left, FIRE and BOMB on the right). The layout uses `viewport-fit=cover` and a canvas scaled from a 480×720 base size.

## Code layout (inside `index.html`)

The script is organized by section comments. Key pieces, in file order:

- **DOM references** (`const canvas = …`, `startBtn`, `scoreEl`, HUD elements, touch controls). Look up new elements here, near the others.
- **Storage keys** (`STORAGE_BEST`, `STORAGE_MUTE`, `STORAGE_DIFF`, `STORAGE_REDUCED`). Use the `skyfighter.*` prefix for any new key.
- **Adaptive performance** (`trackPerf`): samples frame time and steps a `quality` level (0 full, 1 medium, 2 low).
- **Tuning tables**:
  - `DIFFICULTIES`: spawn rates, enemy speed and fire chance, starting lives and bombs, boss HP multiplier.
  - `KILLS_PER_STAGE`: kills needed to advance a stage.
  - `THEMES`: sky palettes that cycle per stage.
  - `ENEMY_TYPES`: `grunt`, `interceptor` (weaving), `tank` (multi-hit). Each has `minStage`, which gates when it spawns.
  - `BOSSES`: name, colors, and `hpMul`. `BOSS_PATTERNS` holds the attack patterns.
- **Audio**: `ensureAudio`, `beep`, `noiseBurst`, and the `sfx` helpers. Audio is created lazily on the first user gesture. Browsers block audio before that.
- **HUD and feedback**: `renderHearts`, `renderBombs`, `flashHit`, `showComboPop`, `showStageBanner`, `screenShake`, `announce` (screen-reader live region).
- **Input**: keyboard (`keys`), touch joystick and buttons, and mouse/pointer on the canvas.
- **Game state**: `resetGame`, `update(dt)`, `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`, `applyPowerUp`, `useBomb`, `fireWeapon`, `addScore` and the kill-chain (`breakChain`), `loseLife`, `killBoss`, `endGame`, `togglePause`.
- **Rendering**: `draw()`, `drawPlane`, `drawBossShip`, `drawClouds`. Clouds and the sun are parallax layers.
- **Loop**: `loop(now)` uses `requestAnimationFrame`, clamps `dt` to 0.05 s, and calls `update` then `draw` only while running and not paused.

Everything is inside one IIFE, so nothing is global. Don't add globals. Expose new behavior through the existing functions and the DOM.

## Conventions

- Keep it a single self-contained file unless the user asks for a split. Don't add a build step, bundler, or dependencies without asking.
- Keep the existing style: `'use strict'`, `const` by default, 2-space indentation, single quotes, and short section comments with `// ---------- Name ----------`.
- Use the existing tuning tables for balance changes instead of hard-coding numbers in the update loop.
- Respect `reducedMotion` for any new screen shake, flashes, or animation. Respect `muted` for any new sound.
- Keep the touch controls working when you add input. Test with `pointer`/`touch` events, not only keyboard.
- Accessibility: announce important state changes (stage, game over, pause) through `announce()`.

## Persistence

- Best score, mute, difficulty, and reduced-motion preference are saved to `localStorage`. Wrap storage access so the game still runs if storage is blocked or throws (private windows, disabled site data).

## Git workflow

- Development branch for this task: `claude/gifted-tesla-fniz2o`. Commit and push there. Don't push to `main` without explicit permission.
- Write clear commit messages that describe the gameplay or code change.
- Don't open a pull request unless the user asks for one.
