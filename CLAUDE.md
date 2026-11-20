# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo contains **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs in the browser. The entire game is a single self-contained file:

- `index.html` (~1,670 lines): inline `<style>` for layout and HUD, markup for the canvas, HUD, overlays and touch controls, and one inline `<script>` IIFE with all game logic, rendering, audio and input.
- `README.md`: a one-line title only.

There is no build step, no package manager, no bundler, no tests, and no external dependencies. The only runtime dependencies are browser APIs (Canvas 2D, Web Audio, `localStorage`, `matchMedia`).

## Running and verifying

- Open `index.html` directly in a browser, or serve the directory with any static server (for example `python3 -m http.server`).
- There is no test suite or linter configured. Verify changes by loading the page and playing it, ideally on both desktop and a phone-sized viewport, since touch controls are part of the game.

## Code layout inside `index.html`

The script is wrapped in an IIFE. Rough order of sections:

1. **DOM references and storage keys**: `STORAGE_BEST`, `STORAGE_MUTE`, `STORAGE_DIFF`, `STORAGE_REDUCED` (all prefixed `skyfighter.`).
2. **Adaptive performance**: `trackPerf` drops the `quality` level (0 full, 1 medium, 2 low) when frame times stay high, and raises it back when they recover.
3. **Tuning data**: `DIFFICULTIES` (spawn rates, enemy speed and fire chance, lives, bombs, boss HP multiplier), `KILLS_PER_STAGE`, `THEMES` (sky palettes per stage), `ENEMY_TYPES` (grunt, interceptor, tank), `BOSSES` (six bosses, cycled by stage).
4. **Audio**: `ensureAudio`, `beep`, `noiseBurst`, plus SFX tables such as `EXPLODE_SFX`. Audio is created lazily on the first user gesture.
5. **HUD and feedback**: `renderHearts`, `renderBombs`, `flashHit`, `flashBomb`, `showComboPop`, `showStageBanner`, `screenShake`, and `announce` for screen-reader text via the `srLive` live region.
6. **Input**: keyboard (`keydown`/`keyup` into a `keys` map keyed by `e.code`), a virtual joystick and fire/bomb buttons for touch, and mouse aiming on the canvas.
7. **Game state and logic**: `resetGame`, `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`, `applyPowerUp`, `fireWeapon`, `useBomb`, `update(dt)`, `loseLife`, `endGame`, `togglePause`, and the boss attack patterns (`BOSS_PATTERNS` and the `fire*` helpers).
8. **Rendering**: `draw()` and the drawing helpers (`drawPlane`, `drawBossShip`, `drawClouds`).
9. **Main loop**: `loop(now)` clamps `dt` to 50 ms, updates when running and not paused, then draws. The start button wires up `resetGame`.

### Controls

- Keyboard: arrow keys or WASD to move, `Space` to fire, `X` or Shift to drop a bomb, `P` or `Escape` to pause.
- Touch: virtual joystick, fire button, bomb button.
- Mouse: move to aim, hold to fire.

## Conventions

- Keep the game as a single file unless the user asks for a split. Put new CSS in the existing `<style>` block and new logic in the existing IIFE.
- Use the existing `const` tuning tables for balance numbers rather than hard-coding them in logic.
- Persist player settings only through the `skyfighter.*` `localStorage` keys, and wrap storage reads and writes so the game still works when storage is unavailable (private windows, blocked site data).
- Respect the reduced-motion setting: new screen shake, flashes or animations should go through `screenShake` or check `reducedMotion`.
- Keep HUD and touch controls usable at phone width. The page is locked to the viewport (`overflow: hidden`, `touch-action: none`), so avoid changes that add scrolling.
- Accessibility text for state changes should go through `announce()`.

## Git

- Development branch for this session: `claude/gifted-tesla-77m8ms`. Push there with `git push -u origin <branch>`; do not push to other branches without permission.
- Do not open pull requests unless the user asks.
