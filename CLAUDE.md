# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a small, dependency-free static web project. Its only real content is **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs in a single HTML5 `<canvas>` page.

| File | Purpose |
| --- | --- |
| `index.html` | The entire game: markup (~360 lines), CSS in a `<style>` block (lines 10–359), and one inline `<script>` IIFE (lines 422–1667). |
| `README.md` | Placeholder title only. |

There is no build step, no package manager, no test suite, and no linter configured. Nothing needs to be installed.

## Running and checking changes

- Open `index.html` directly in a browser, or serve the folder with any static server (e.g. `python3 -m http.server`).
- Verify changes by playing the game in a browser. Check both desktop (keyboard/mouse) and touch layouts when touching input or HUD code.
- Keep the page working as a single file. Do not split it into modules or add a bundler unless the user asks.

## Code layout inside `index.html`

The script is one `(function () { 'use strict'; ... })();` closure. Everything is `const`/`let` inside it, so nothing leaks onto `window`. Keep new code inside that closure.

Major sections, in file order (search for the `// ----------` comments and function names):

1. **DOM references**: every `getElementById` lookup is grouped at the top.
2. **Storage keys**: `STORAGE_BEST`, `STORAGE_MUTE`, `STORAGE_DIFF`, `STORAGE_REDUCED`, all prefixed `skyfighter.`.
3. **Adaptive performance**: `trackPerf` samples frame time and sets `quality` (0 full, 1 medium, 2 low).
4. **Tuning tables**:
   - `DIFFICULTIES`: spawn rates, enemy speed and fire chance, starting lives and bombs, boss HP multiplier.
   - `KILLS_PER_STAGE`: kills needed to advance a stage.
   - `THEMES`: sky gradients and cloud/sun colors, picked by stage with `% THEMES.length`.
   - `ENEMY_TYPES`: `grunt`, `interceptor` (weaving), `tank` (4 HP, unlocked at stage 2).
   - `BOSSES`: six bosses, picked by `(stage - 1) % BOSSES.length`, each with colors and `hpMul`.
   - `BOSS_PATTERNS`: bullet-pattern functions (`fireRadial`, `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`, `fireCross`, `fireHomingSeeded`, `fireDoubleRing`).
5. **Audio**: `ensureAudio`, `beep`, `noiseBurst`, plus the `EXPLODE_SFX` map. Audio is created lazily on the first user gesture.
6. **HUD and effects**: `renderHearts`, `renderBombs`, `flashHit`, `flashBomb`, `showComboPop`, `showStageBanner`, `screenShake`.
7. **Layout**: `BASE_W = 480`, `BASE_H = 720`. `resize()` scales the canvas to fit the window.
8. **Input**: keyboard (`keys` map, keydown/keyup), mouse via `canvasPointFromEvent`, and touch via the joystick and FIRE/BOMB buttons.
9. **Game state**: `resetGame`, `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`, `applyPowerUp`, `fireWeapon`, `useBomb`, `addScore`/`breakChain` (kill-chain multiplier), `loseLife`, `endGame`, `togglePause`.
10. **Update and draw**: `update(dt)` advances the simulation, `updateBoss(dt)` drives boss phases, and `draw()` renders everything. `drawPlane`, `drawBossShip`, and `drawClouds` are the sprite helpers.
11. **Main loop**: `loop(now)` clamps `dt` to 0.05 s, then calls `trackPerf`, `update`, and `draw` while the game is running and not paused. It is driven by `requestAnimationFrame`.

### Conventions used in this file

- Frame-rate-independent movement: every update takes `dt` in seconds. Keep it that way.
- Persistence goes through `localStorage` wrapped in plain reads and writes. Keep the reads tolerant of missing values, as they are now (`|| 0`, `|| 'normal'`).
- Accessibility: `announce()` writes to the `srLive` live region. `reducedMotion` (defaulting to the `prefers-reduced-motion` media query) shrinks screen shake and skips the hit and bomb flashes. New flashing or shaking effects should respect this flag.
- Controls: Space or mouse fires, arrows/WASD move, X or Shift bombs, P or Esc pauses. Update `controlsHint` in the overlay HTML if you change bindings.
- Enemies, bosses, and power-ups are plain objects in arrays that are filtered each frame. Follow that pattern rather than introducing classes.

## Editing guidelines

- Match the existing style: 2-space indent, single quotes in JS, `const` by default, arrow functions for callbacks, short section comments using `// ---------- Name ----------`.
- Tune balance by changing the tables near the top of the script rather than hard-coding numbers inside update logic.
- Keep the file self-contained. Do not add external CDN scripts, fonts, or assets without an explicit request.
- The mobile layout relies on `viewport-fit=cover` and the touch-control overlay; test it after any CSS change.

## Git workflow

- Develop on the branch the session names (currently `claude/gifted-tesla-pmdwle`); do not push to `main`.
- Write descriptive commit messages in the imperative mood, with a summary line and a short body for non-trivial changes.
- Do not open pull requests unless the user asks.
