# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo contains **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs as a single self-contained HTML5 canvas page.

- `index.html` (~1,670 lines): the entire game. CSS is in a `<style>` block, markup is at the top of `<body>`, and all game logic is in one inline `<script>` wrapped in an IIFE with `'use strict'`.
- `README.md`: a placeholder stub. There is no other documentation.

There is no build step, no package manager, no framework, no test suite, and no linter configuration. Do not add tooling (npm, bundlers, frameworks) unless the user asks for it.

## Running the game

Open `index.html` directly in a browser, or serve the directory with any static server, for example `python3 -m http.server`. No install is needed.

## Verifying changes

Because there are no automated tests, verify changes by loading the page in a browser and playing through the affected behavior. Check at least:

- The page loads with no console errors.
- Desktop controls and touch controls both still work, if the change touches input.
- The overlay, HUD, and pause menu still render correctly.
- Changes that affect score, lives, or bombs behave correctly across all three difficulties.

A headless Chromium with Playwright is available in the cloud environment (`chromium.launch()` with no options). Use it for scripted checks when useful.

## Code structure inside `index.html`

The inline script is organized roughly in this order. Keep new code in the matching section.

1. **DOM references**: `const xEl = document.getElementById(...)` at the top of the IIFE.
2. **Storage keys**: `STORAGE_BEST`, `STORAGE_MUTE`, `STORAGE_DIFF`, `STORAGE_REDUCED` (all prefixed `skyfighter.`).
3. **Config tables**:
   - `DIFFICULTIES`: `easy` / `normal` / `hard`, with `spawnBase`, `spawnMin`, `enemySpeed`, `enemyShootChance`, `lives`, `bombs`, `bossHpMul`.
   - `KILLS_PER_STAGE` (18): kills needed before a boss spawns.
   - `THEMES`: six sky palettes that cycle per stage.
   - `ENEMY_TYPES`: `grunt`, `interceptor` (weaving), `tank` (4 HP). Each has a `minStage` gate.
   - `BOSSES`: six bosses (VANGUARD through TITAN), each with colors and an `hpMul`.
   - `BOSS_PATTERNS`: bullet-pattern functions (radial, aimed spread, columns, spiral, cross, homing, double ring).
4. **Audio**: `ensureAudio()`, `beep()`, `noiseBurst()`, and a `sfx` object. Audio is created lazily on the first user gesture, which browsers require.
5. **UI helpers**: hearts and bombs rendering, flashes, combo pops, stage banner, screen shake, mute and motion toggles, `announce()` for screen readers.
6. **Layout and input**: `resize()` scales a fixed `BASE_W × BASE_H` (480 × 720) logical canvas. Joystick, mouse, and touch handlers live here.
7. **Game state**: `resetGame()` initializes the `player`, arrays of entities, and counters.
8. **Spawning and combat**: `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`, `fireWeapon`, `useBomb`, `applyPowerUp`, `addScore` / `breakChain` (kill-chain multiplier).
9. **Update loop**: `update(dt)` advances the simulation. `updateBoss(dt)`, `killBoss()`, `loseLife()`, and `endGame()` handle the major state transitions.
10. **Rendering**: `draw()` and its helpers (`drawPlane`, `drawBossShip`, `drawClouds`). Rendering is separate from update logic. Keep it that way.
11. **Main loop**: `loop(now)` computes a delta-time `dt` in seconds, clamped to 0.05 s, then calls `trackPerf`, `update`, and `draw`. It only updates while the game is running and not paused.

## Game mechanics reference

- **Controls (keyboard):** Arrow keys or WASD to move. Space or hold to fire. X, Shift, or the bomb button to drop a smart bomb. P or Escape to pause.
- **Controls (touch and mouse):** on-screen joystick, fire button, and bomb button.
- **Weapons:** `vulcan` (spread shots) and `laser` (piercing beams). Each has three power levels.
- **Power-ups:** `life` (extra life), `bomb` (adds a bomb), `weapon` (raises the weapon level until level 3; at level 3 it switches to the other weapon and resets the level to 1).
- **Smart bombs:** clear the screen and deal damage to bosses. The count depends on difficulty.
- **Stages:** after 18 kills, a boss fight starts. Defeating a boss increments `stage`. The sky theme is `THEMES[(stage - 1) % THEMES.length]`, so it cycles through all six.
- **Persistence:** best score, mute, difficulty, and reduced-motion preference are stored in `localStorage` under `skyfighter.*` keys. Wrap any new storage access so it doesn't break when storage is unavailable.

## Conventions

- Keep everything in `index.html` unless there is a clear reason to split it. The single-file design is intentional.
- Use `const` and `let`. The file already uses `'use strict'`, so keep it.
- Put tunable numbers in the config tables (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `BOSS_PATTERNS`) rather than hard-coding them in logic.
- Keep update logic and draw logic separate. `draw()` should not mutate game state.
- Preserve accessibility hooks: the `srLive` live region via `announce()`, the reduced-motion setting, and the `prefers-reduced-motion` check. Screen-shake and flash effects should respect reduced motion.
- Preserve mobile support. The `<meta name="viewport">` settings and the touch controls are part of the product.
- Match the existing style: two-space indentation, single quotes in JS, and short helper functions.

## Git workflow

- Develop on the branch the user designates. Do not push to other branches without permission.
- Write clear commit messages describing the gameplay or code change.
- Do not open a pull request unless the user asks for one.
