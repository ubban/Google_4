# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a small, dependency-free repo. Its only game is **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs as a single self-contained HTML5 canvas page.

| File | Purpose |
| --- | --- |
| `index.html` | The entire game: markup, CSS (`<style>`), and JavaScript (`<script>`), about 1,670 lines. |
| `README.md` | Placeholder title only. |
| `CLAUDE.md` | This file. |

There is no build step, package manager, test suite, or linter configured.

## Running the game

- Open `index.html` directly in a browser, or serve the folder with any static server, for example `python3 -m http.server` and then visit `http://localhost:8000`.
- Nothing needs to be installed.

## Architecture of `index.html`

All game code lives in one IIFE (`(function () { 'use strict'; ... })();`) inside the single `<script>` block. Keep new code inside that scope rather than adding globals.

Major sections, in file order:

1. **DOM references**: `const xxxEl = document.getElementById(...)` for the canvas, HUD, overlay, touch controls, and boss bar.
2. **Storage keys**: `STORAGE_BEST`, `STORAGE_MUTE`, `STORAGE_DIFF`, `STORAGE_REDUCED`, all prefixed `skyfighter.`. Changing a key orphans saved player data.
3. **Adaptive performance**: `quality` (0 full, 1 medium, 2 low) is adjusted by `trackPerf(dt)` from a rolling average of frame times.
4. **Tuning tables**:
   - `DIFFICULTIES` (`easy`, `normal`, `hard`): spawn rates, enemy speed, shoot chance, starting lives and bombs, and boss HP multiplier.
   - `KILLS_PER_STAGE`: kills needed to advance a stage.
   - `THEMES`: sky gradients and palette, selected by `(stage - 1) % THEMES.length`.
   - `ENEMY_TYPES`: `grunt`, `interceptor` (weaves), `tank` (4 HP). Each has `minStage`, which gates when it spawns.
   - `BOSSES`: six boss designs, selected by `(stage - 1) % BOSSES.length`, each with `hpMul`.
5. **Audio**: created lazily by `ensureAudio()`, which the browser's user gesture requires. Mute state is persisted.
6. **Rendering helpers**: `drawPlane`, `drawBossShip`, `drawClouds`, and the `draw()` function. Drawing is canvas 2D only.
7. **Entities and gameplay**: `spawnEnemy`, `spawnBoss`, `spawnExplosion`, `useBomb`, `update(dt)`, `updateBoss(dt)`.
8. **Pause**: `togglePause()`.
9. **Main loop**: `loop(now)` drives `update` then `draw` each frame, with a clamped `dt`.
10. **Input**: keyboard, touch joystick and fire/bomb buttons, and mouse, wired near the bottom of the script.

### Controls

- Move: arrow keys or WASD (keyboard), mouse (the plane follows the cursor), or the on-screen joystick (touch).
- Fire: hold Space, or the fire button on touch. Clicking the canvas with the mouse also fires.
- Smart bomb: `X` or Shift, or the bomb button. Bombs clear the screen and damage bosses.
- Pause: `P` or `Escape`.
- Mute and reduced-motion are toggle buttons in the UI (`muteBtn`, `motionBtn`).

### Persistence

Only these values are stored in `localStorage`: best score, mute, difficulty, and reduced-motion preference. Every access is a direct `localStorage` call with no `try/catch`, so the page can throw in browsers where storage is blocked. If you touch these lines, consider wrapping them.

## Conventions

- **Single file.** Keep HTML, CSS, and JS in `index.html` unless the user asks for a split. Do not introduce a bundler, framework, or npm dependencies without being asked.
- **Vanilla JS, ES2015+ syntax** as already used: `const`/`let`, arrow functions, template literals. Match the existing style: 2-space indentation, single quotes, semicolons.
- **Tuning lives in the constant tables** at the top of the script, not inline in the update loop. Add new enemies, bosses, or themes by extending those tables.
- **Time-based updates.** Movement and timers use the `dt` passed to `update`. Do not hard-code per-frame increments.
- **Accessibility hooks.** The `body.reduced-motion` class disables the vignette, combo pop, stage banner, overlay and icon animations, and hit/bomb flashes via CSS. Any new flashing or animated effect should be covered by the same class. Announce important state changes through `announce()`, which feeds the `srLive` live region.
- **Touch and desktop.** Any new control needs both a keyboard/mouse path and a touch path, and layouts must work at phone width (`viewport-fit=cover` and safe-area insets are already used).

## Verifying changes

There are no automated tests. Before reporting a change as working:

1. Load `index.html` in a browser and check the console for errors.
2. Play through at least one stage transition and one boss fight, since they are the most state-heavy paths.
3. Check pause/resume, bomb use, and game-over → restart.
4. If the change touches layout or the HUD, check a narrow (phone-width) viewport as well as desktop.

If you cannot run a browser, say so explicitly and limit claims to what you verified by reading the code.

## Git and workflow

- Default working branch for automated work: `claude/gifted-tesla-iohesq`. Do not push to other branches without explicit permission.
- Commit messages should describe the gameplay or code change in the imperative mood, with a short body when the change is non-trivial.
- Do not create pull requests unless explicitly asked.
- Do not commit local secrets or environment files; this repo currently has none.

## Things to avoid

- Do not add external CDN scripts, fonts, or assets. The game is intentionally self-contained.
- Do not rename storage keys or change the stored value format without a migration path, or players lose their best score and settings.
- Do not rewrite the file wholesale for small changes. Make targeted edits so the diff stays reviewable.
