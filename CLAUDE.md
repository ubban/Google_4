# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo is **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs in the browser. The entire game lives in one file, `index.html` (~1,670 lines): HTML markup, an inline `<style>` block, and one inline `<script>` that wraps everything in an IIFE. `README.md` is a placeholder.

There is no build step, no package manager, no bundler, no dependencies, and no test suite. Open `index.html` in a browser to play.

## Repository layout

- `index.html`: the whole game (markup, CSS, JS).
- `README.md`: minimal placeholder.
- `CLAUDE.md`: this file.

## Code structure of `index.html`

Within the main `<script>` IIFE, the code is organized roughly in this order:

1. **DOM references**: `canvas`, overlay and HUD elements, touch controls, and buttons.
2. **Storage keys**: `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion`.
3. **Config tables**: `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES` (per-stage sky palettes), `ENEMY_TYPES` (grunt, interceptor, tank), and `BOSSES` (six bosses with hull colors and HP multipliers).
4. **Audio**: a lazily created `AudioContext` (`ensureAudio`), with `beep`, `noiseBurst`, and a `sfx` map. Audio starts only after a user gesture.
5. **HUD and effects**: `renderHearts`, `renderBombs`, `flashHit`, `showComboPop`, `showStageBanner`, `screenShake`.
6. **Layout**: `resize()` fits a fixed logical canvas of **480×720** into the window, scaled up to 1.6× max.
7. **Input**: keyboard state in `keys`, plus mouse, touch joystick, and fire/bomb buttons.
8. **Game logic**: `resetGame`, `update(dt)`, `updateBoss(dt)`, `spawnEnemy`, `spawnBoss`, `spawnExplosion`, `useBomb`, `togglePause`.
9. **Rendering**: `draw()`, `drawPlane`, `drawBossShip`, `drawClouds`.
10. **Main loop**: `loop(now)` drives `update` and `draw` each frame.

Frame timing is tracked by `trackPerf(dt)`. Keep frame-time work cheap, since this is a real-time canvas game.

## Controls

- Mouse: move to steer, click to fire.
- Arrow keys or WASD: move. Space: fire. X or Shift: smart bomb. P or Esc: pause.
- Touch: drag left to fly, tap FIRE, tap BOMB.

Keyboard handling lives in the global `keydown` and `keyup` listeners. Space is `preventDefault`-ed to stop page scrolling.

## Game rules and tuning

- Difficulty (`easy`, `normal`, `hard`) changes spawn rate, enemy speed, enemy fire chance, starting lives and bombs, and boss HP multiplier. Tune values in `DIFFICULTIES`.
- Each stage has a theme from `THEMES`. Reaching `KILLS_PER_STAGE` kills (currently 18) triggers the stage's boss warning and boss fight.
- Tank enemies appear from stage 2 (`minStage`). Grunts and interceptors appear from stage 1.
- Weapons: Vulcan and Laser, each with 3 power levels. Smart bombs clear the screen and damage bosses.
- Kill-chain score multiplier and power-ups (weapon level, bomb, extra life) are part of the core loop.
- `reducedMotion` (from the OS preference, overridable in-game) turns off or reduces screen shake and similar effects.

## Persistence

Settings and best score use `localStorage` with the `skyfighter.*` keys above. Every access should be safe when storage is blocked. Keep any new persisted value under the `skyfighter.` prefix and document it here.

## Conventions for changes

- Keep the game as a single self-contained `index.html` unless the user asks for a restructure. Do not add a build system or external dependencies without asking.
- Preserve the existing style: 2-space indentation, `const`/`let`, and small named functions inside the IIFE.
- Put new tunable numbers in the existing config tables (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `THEMES`) rather than hard-coding them in logic.
- Respect the logical 480×720 coordinate space. Do not hard-code screen pixels.
- Keep the UI usable on phones: the viewport meta disables zoom, and touch controls must keep working.
- Keep accessibility hooks intact (the `srLive` live region, `announce()`, and reduced-motion handling).

## Verifying changes

There are no automated tests or linters. To verify a change:

1. Open `index.html` in a browser, or serve the directory with any static server.
2. Start a game on each difficulty and check the HUD, lives, bombs, and pause.
3. Reach a stage transition and a boss fight, then confirm the boss health bar and the bomb damage.
4. Check the touch controls with a mobile viewport, and check the layout at narrow widths.
5. Confirm that best score, mute, difficulty, and reduced-motion settings persist across reloads.

For JavaScript syntax checks without a browser, extract the inline script and run `node --check` on it.

## Git

- Development branch for this work: `claude/gifted-tesla-vp7mi8`.
- Use clear, descriptive commit messages that describe the gameplay or code change.
