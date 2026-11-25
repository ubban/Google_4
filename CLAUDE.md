# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Google_4 is a small static web project. Its only substantive content is **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs in the browser on a 2D canvas.

Files:

- `index.html` (~1,670 lines): the entire game. Inline CSS (`<style>`, roughly lines 10–359), markup with a `<canvas id="gameCanvas">` and HUD/overlay elements, and one inline `<script>` (roughly lines 422–1667) containing the game as a single IIFE.
- `README.md`: a stub. It contains only a title and a timestamp.

There is no `package.json`, bundler, framework, test suite, linter, or CI configuration.

## Running and verifying

- Open `index.html` directly in a browser, or serve the directory with any static file server (for example `python3 -m http.server`).
- There is no build step and no automated test command. Verify changes by playing the game in a browser. For headless checks, Playwright with the pre-installed Chromium works.
- Keep the game a single self-contained HTML file unless the user asks for a split. Do not add a build system or dependencies without asking.

## Code layout inside `index.html`

The script is one `(function () { 'use strict'; ... })();` block. Its sections, in order:

1. **DOM references**: `const x = document.getElementById(...)`.
2. **Persistence keys**: `STORAGE_BEST`, `STORAGE_MUTE`, `STORAGE_DIFF`, `STORAGE_REDUCED`, all prefixed `skyfighter.`.
3. **Adaptive quality**: `trackPerf` lowers the `quality` level (0 full, 1 medium, 2 low) when frame times are slow and raises it when they are fast.
4. **Tuning tables**: `DIFFICULTIES` (spawn rates, enemy speed and fire chance, lives, bombs, boss HP multiplier), `KILLS_PER_STAGE`, `THEMES` (background palettes cycled per stage), `ENEMY_TYPES`, `BOSSES`.
5. **Audio**: a lazily created `AudioContext` (`ensureAudio`), with `beep`, `noiseBurst`, and a named `sfx` set.
6. **HUD helpers**: hearts, bombs, combo pop, stage banner, screen shake, flashes.
7. **Input**: keyboard (`keys` map), touch joystick and fire/bomb buttons, and mouse aim on the canvas.
8. **Game state and logic**: `resetGame`, `update(dt)`, spawning (`spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`), weapons (`fireWeapon`, `applyPowerUp`), bomb (`useBomb`), scoring and chain (`addScore`, `breakChain`), boss attack patterns (`BOSS_PATTERNS`, `updateBoss`, `killBoss`), life loss and game over (`loseLife`, `endGame`), pause (`togglePause`).
9. **Rendering**: `draw()` plus sprite helpers (`drawPlane`, `drawBossShip`, `drawClouds`).
10. **Main loop**: `loop(now)` runs `update` and `draw` only when the game is running and not paused. `dt` is clamped to 50 ms.

Conventions to keep:

- The game uses a fixed logical resolution of 480×720 (`BASE_W`, `BASE_H`), scaled by `resize()`. Gameplay coordinates should stay in logical units.
- Persistent state uses `localStorage` with the `skyfighter.` prefix. Keep new keys under that prefix.
- Gameplay constants belong in the tuning tables near the top of the script, not inline in logic.
- Accessibility features exist and should be preserved: `prefers-reduced-motion` handling, the `srLive` ARIA live region for announcements (use `announce(text)`), and touch controls that call `preventDefault` where needed.

## Editing guidelines

- Make small, targeted edits. The file is large, so read the relevant section before changing it.
- Do not reformat or restructure unrelated code.
- Keep the code in vanilla JavaScript and CSS, with no external dependencies.
- After changing gameplay, check the affected difficulty, stage, and boss paths by playing, since there are no automated tests.

## Git

- Develop on the branch assigned to the session. Do not push to other branches without explicit permission.
- Commit only when asked. Write clear commit messages that describe the gameplay or code change.
