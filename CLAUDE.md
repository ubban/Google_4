# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo is **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs entirely in the browser. The whole game is a single self-contained file:

- `index.html` (~1,670 lines): inline `<style>` (lines ~10–359), markup (HUD, overlay, touch controls), and one IIFE `<script>` (starts ~line 422) containing all game logic, rendering, audio, and input.
- `README.md`: a placeholder. It contains no real project documentation.

There is no build step, no bundler, no package manager, no test suite, and no linter. Do not add any of these unless the user asks.

## Running the game

Open `index.html` directly in a browser, or serve the directory with any static server (for example `python3 -m http.server`). Nothing needs to be installed.

## Architecture of `index.html`

Everything below lives inside the single IIFE in the `<script>` block.

- **Canvas.** Rendering uses a 2D canvas (`gameCanvas`) at a logical size of `BASE_W = 480`, `BASE_H = 720`. `resize()` scales the canvas to fit the window, capped at 1.6x.
- **Main loop.** `loop(now)` runs on `requestAnimationFrame`. `dt` is clamped to 0.05 s. When the game is running and not paused, it calls `update(dt)` and then `draw()`. `trackPerf(dt)` adjusts the `quality` level (0 full, 1 medium, 2 low) when frame times are poor.
- **Config tables** (top of the script): `DIFFICULTIES` (easy / normal / hard, with spawn rates, enemy speed, lives, bombs, and boss HP multipliers), `KILLS_PER_STAGE`, `THEMES` (background palettes, one per stage, cycled), `ENEMY_TYPES` (grunt, interceptor, tank, with `minStage` gating), and `BOSSES` (six boss designs, cycled by stage).
- **Game state.** Module-level `let` variables in the IIFE. `resetGame()` initializes a new run. Stage progression comes from kill counts (`KILLS_PER_STAGE`).
- **Update and draw.** `update(dt)` handles movement, firing, enemy spawning, collisions, power-ups, bombs, and bosses. `draw()` and helpers such as `drawPlane`, `drawBossShip` render the frame. `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`, `spawnExplosion`, and `fireWeapon` are the main spawn and effect entry points.
- **Audio.** Web Audio API, created lazily by `ensureAudio()` (browsers require a user gesture first). Sound effects go through `beep()`, `noiseBurst()`, and the `sfx` map. Respect the `muted` flag.
- **HUD and overlay DOM.** `renderHearts`, `renderBombs`, `showComboPop`, `showStageBanner`, and the boss bar are updated from game state. Overlay screens (start, pause, game over) are driven by `overlay`, `overlayTitle`, `overlaySub`, and the stats row.
- **Input.** Keyboard: arrows/WASD move, Space fires, X or Shift bombs, P or Esc pauses. Mouse: the canvas tracks `mouseTarget`, and mouse-down fires. Touch: a virtual joystick (`moveZone`), a fire button, and a bomb button, shown when `isTouchDevice` is true.
- **Persistence.** `localStorage` only, with keys `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, and `skyfighter.reducedMotion`. Every read must tolerate missing or blocked storage.
- **Accessibility.** `prefers-reduced-motion` is detected at startup and can be toggled with `motionBtn`. Reduced motion shrinks screen shake (`screenShake`) and skips hit and bomb flashes (`flashHit`, `flashBomb`). The `srLive` live region carries announcements (`announce()`).

## Conventions

- **Keep it a single file.** Put new CSS in the existing `<style>` block and new JS in the existing IIFE, unless the user asks for a split. Do not introduce external dependencies or CDN scripts.
- **Preserve the style of the file.** Two-space indentation, single quotes in JS, `const` by default, and tuning values kept in the config tables near the top of the script rather than inlined in logic.
- **Keep gameplay tuning data-driven.** New enemies, bosses, themes, and difficulty settings should be added as entries in the existing tables, not as special cases in `update()` or `draw()`.
- **Mobile first.** Keep the layout working at phone width. The page disables zoom and scroll (`overflow: hidden`, `touch-action: none`), so any new UI must fit inside the viewport and respect `env(safe-area-inset-*)`.
- **Performance.** Per-frame code runs at 60 fps. Avoid allocations inside `update()` and `draw()` where practical, and respect the `quality` level when adding particles or other effects (see `spawnExplosion` for the pattern).

## Verifying changes

Because there are no automated tests:

1. Load `index.html` in a browser (or the Playwright-based browser available in cloud sessions) and check the console for errors.
2. Play through at least one stage on the affected difficulty. When touching bosses or stage transitions, reach the boss fight.
3. Check the pause, mute, reduced-motion, and difficulty toggles if the change could affect them.
4. Check the layout at a narrow (phone-width) viewport and in landscape.

## Git

- Development branch for this work: `claude/gifted-tesla-dq8ov5`. Push with `git push -u origin <branch>`.
- Commit messages should be short and imperative, describing the gameplay or UI change (see the existing `c4dc8f6` commit for tone).
