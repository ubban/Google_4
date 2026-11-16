# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo contains **Sky Fighter**, a single-page, Raiden-style vertical shoot-'em-up built on an HTML5 `<canvas>`. There is no build step, no package manager, and no test suite.

Files:

- `index.html` (~1,670 lines): the entire game. CSS is in a `<style>` block, markup in the body, and all game logic in one inline `<script>` wrapped in an IIFE with `'use strict'`.
- `README.md`: a placeholder. It currently holds a stray timestamp line rather than real docs.

## Running and testing

- Open `index.html` directly in a browser, or serve the directory with any static server (for example `python3 -m http.server`).
- There are no tests, linters, or build commands. Verify changes by running the game in a browser and playing through the affected path. Check desktop keyboard controls and, when touch-related code is touched, the touch controls.

## Code structure (inside `index.html`)

The script is organized by `// ---------- Section ----------` comments. Major sections, in source order:

- **DOM references and constants**: cached element lookups, `STORAGE_*` keys, `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES` (per-stage sky palettes), `ENEMY_TYPES` (grunt, interceptor, tank), `BOSSES` (six boss definitions).
- **Adaptive performance**: `trackPerf` samples frame times and steps a `quality` level (0 full, 1 medium, 2 low).
- **Audio**: `ensureAudio`, `beep`, `noiseBurst`, and the `EXPLODE_SFX` mapping. Audio is synthesized with WebAudio, with no audio files.
- **HUD and feedback**: `renderHearts`, `renderBombs`, `showComboPop`, `showStageBanner`, `screenShake`, `flashHit`/`flashBomb`.
- **Layout and input**: `resize` (fixed logical size `BASE_W` x `BASE_H` = 480 x 720, scaled to the viewport), joystick handling for touch, mouse/touch pointer mapping via `canvasPointFromEvent`, and keyboard handling.
- **Game state**: `resetGame`, `update(dt)` (main simulation step), `applyPowerUp`, `useBomb`, `fireWeapon`, `addScore`/`breakChain` (kill-chain multiplier), `loseLife`, `endGame`, `togglePause`.
- **Enemies and bosses**: `spawnEnemy`, `spawnBoss`, `updateBoss`, `killBoss`, and bullet pattern helpers (`fireRadial`, `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`, `fireCross`, `fireHomingSeeded`, `fireDoubleRing`) consumed by the `BOSS_PATTERNS` array.
- **Rendering**: `drawPlane`, `drawBossShip`, `drawClouds`, and `draw()`. Drawing is plain Canvas 2D primitives.
- **Main loop**: `loop(now)` clamps `dt` to 50 ms, runs `update` and `draw` only while `running && !paused`, then re-queues with `requestAnimationFrame`.

## Conventions

- Keep everything in `index.html`. Do not add a build system, bundler, or framework without an explicit request. Keep it a single file that opens directly in a browser.
- Keep the IIFE and `'use strict'` wrapper. Game state lives in `let` variables inside the closure, not on `window`.
- Tuning values (difficulty, enemy stats, boss HP multipliers, spawn rates) belong in the existing constant tables at the top of the script. Prefer adjusting those over adding magic numbers into logic.
- Persist user settings only through the `STORAGE_*` keys (`skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion`). Use the same `'1'`/`'0'` string convention for booleans.
- Respect the reduced-motion setting (`reducedMotion`, toggled from the HUD). It exists to tone down screen shake, flashes, and pulses. New motion-heavy effects should honor it too.
- Announce important state changes for screen readers through the `announce()` helper and the `srLive` live region.
- Game-state changes that affect both desktop and touch should be wired in both input paths.

## Controls (for reference when testing)

- Move: arrow keys or `A`/`D`/`W`/`S` on desktop, or the on-screen joystick on touch devices.
- Fire: `Space` (hold), or the fire button on touch devices and mouse hold on the fire button.
- Bomb: `X` or either `Shift` key, or the bomb button on touch devices.
- Pause: `P` or `Escape`.
- The mute, reduced-motion, and difficulty toggles are in the HUD/overlay.

## Git

- Development branch for this work: `claude/gifted-tesla-jamcwp`.
- Keep commits focused. Describe the gameplay or UI change in the message, not just the file touched.
