# CLAUDE.md

Guidance for working in this repository.

## What this is

**Sky Fighter** — a Raiden-style vertical shoot-'em-up that runs entirely in the browser. The whole game is one self-contained file, `index.html` (~1,670 lines): inline `<style>`, the markup, and one inline `<script>` IIFE. There are no external dependencies, no bundler, no package manager, and no test suite.

`README.md` is a placeholder and carries no project documentation.

## Running and checking changes

- Open `index.html` directly in a browser, or serve the directory with any static server (e.g. `python3 -m http.server`).
- There is no build step, lint config, or automated test. Verify changes by playing the game in a browser: start a run, exercise the feature you touched, and check the console for errors.
- Test both input paths when you touch controls: keyboard and touch (the joystick and fire/bomb buttons appear on touch devices).

## Code layout (inside `index.html`)

- **`<style>`** — all CSS, including the HUD pills, the glassmorphic overlay card, touch controls, and the `.reduced-motion` class.
- **`<body>`** — the `<canvas id="gameCanvas">`, HUD elements, overlay (title / stats / start button), touch controls, and a `#srLive` `aria-live` region for screen-reader announcements.
- **`<script>`** — wrapped in `(function () { 'use strict'; ... })()`. Sections, in order:
  1. DOM references (`const xxxEl = document.getElementById(...)`).
  2. Persistence keys (`STORAGE_BEST`, `STORAGE_MUTE`, `STORAGE_DIFF`, `STORAGE_REDUCED`) under the `skyfighter.` prefix.
  3. Adaptive quality (`trackPerf`) — drops to lower render quality if frame time stays high.
  4. Data tables: `DIFFICULTIES`, `THEMES` (sky palettes per stage), `ENEMY_TYPES`, `BOSSES`.
  5. Audio — `ensureAudio`, `beep`, `noiseBurst`, and a named-sound table (`EXPLODE_SFX`). Audio is synthesized with WebAudio, so there are no sound files.
  6. HUD/feedback helpers — hearts, bombs, flash, combo pop, stage banner, screen shake.
  7. Layout and input — `resize` (fixed `BASE_W = 480`, `BASE_H = 720` logical canvas scaled to fit), joystick, pointer mapping.
  8. Game state — `resetGame`, `update(dt)`, `draw()`, plus spawning (`spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`) and combat (`fireWeapon`, `useBomb`, `addScore`, `breakChain`, `loseLife`, `endGame`, `togglePause`).
  9. Boss attacks — `BOSS_PATTERNS` (array of bullet-pattern functions such as `fireRadial`, `fireCross`, `fireHomingSeeded`), driven by `updateBoss`.
  10. Rendering — `drawPlane`, `drawBossShip`, `drawClouds`, `draw()`.
  11. `loop(now)` — the `requestAnimationFrame` loop; `dt` is clamped to 50 ms. `startBtn` wiring is at the end.

## Game model

- **Stages:** each stage lasts `KILLS_PER_STAGE` (18) kills, then a boss fights. There are six bosses and six sky themes, cycled per stage.
- **Enemies:** `grunt`, `interceptor` (weaves), and `tank` (4 HP, appears from stage 2).
- **Weapons:** Vulcan / Laser, three power levels, upgraded by weapon power-ups.
- **Bombs:** limited per run; they clear the screen and damage bosses.
- **Power-ups:** `life`, `bomb`, `weapon`. Colors are set in `draw()`.
- **Score:** a kill-chain multiplier (`addScore`, `breakChain`).
- **Difficulty:** `easy` / `normal` / `hard` change spawn rate, enemy speed, enemy fire chance, lives, starting bombs, and boss HP. Persisted via `STORAGE_DIFF`.
- **Best score:** stored in `localStorage` under `skyfighter.bestScore`.

## Controls

| Action | Keyboard | Touch |
| --- | --- | --- |
| Move | Arrow keys or WASD | Left-side joystick |
| Fire (hold) | Space | Fire button |
| Bomb | X or Shift | Bomb button |
| Pause | P or Esc | Pause button |
| Mute / reduced motion | Buttons in the HUD | Buttons in the HUD |

Space's default action is suppressed in `keydown`, so do not remove that `preventDefault` call.

## Conventions

- Keep the game in a single file unless the user asks to split it. Keep all new code inside the existing IIFE; do not add globals.
- Match the surrounding style: `const`/`let`, arrow functions for callbacks, two-space indentation, and short section comments in the form `// ---------- Name ----------`.
- Put tunable numbers in the data tables (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `THEMES`) rather than hard-coding them in logic.
- Entity objects are plain `{ x, y, w, h, ... }` records in arrays that are filtered each frame. Follow that pattern for new entities.
- Persist state only through the `skyfighter.*` localStorage keys, and wrap new reads and writes in a way that tolerates storage being unavailable.
- Keep the page usable on phones: the viewport meta disables zoom, and touch controls must stay reachable. Preserve the `aria-live` announcements for important state changes (stage start, game over, new best).
- Respect `prefers-reduced-motion` (the `reducedMotion` flag) when adding screen shake, flashes, or other motion effects.

## Git

- Development branch: `claude/gifted-tesla-bv6eds`. Commit and push there; do not push to other branches without permission.
- Do not open a pull request unless explicitly asked.
