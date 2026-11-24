# CLAUDE.md

Guidance for working on this repository.

## Project overview

Google_4 is a small static web project. Its only real content is **Sky Fighter**, a
Raiden-style vertical shoot-'em-up written as a single self-contained HTML5 canvas page.

| File        | Purpose                                                        |
|-------------|----------------------------------------------------------------|
| `index.html`| The entire game: inline CSS, markup, and one IIFE `<script>`.  |
| `README.md` | Placeholder title only.                                        |

There is no build step, no package manager, no bundler, and no test suite. Nothing
needs to be installed. Open `index.html` in a browser (or serve the directory with any
static server) to play.

## Repository layout of `index.html`

- **`<head>`**: meta tags, an inline SVG favicon, and the `<style>` block (about lines
  10–359). CSS uses a dark palette (`color-scheme: dark`), a pill-style HUD, and a
  glassmorphic overlay card.
- **`<body>`**: `#container` wrapping `<canvas id="gameCanvas">`, HUD elements, overlay,
  touch controls (`moveZone`, `fireBtn`, `bombBtn`), and buttons such as mute, motion,
  and difficulty.
- **`<script>`** (about lines 422–1665): one `(function () { 'use strict'; ... })();`
  IIFE. Nothing is exposed on `window`; keep it that way unless there is a clear need.

Key sections inside the script, in roughly the order they appear:

1. **Persistence keys**: `STORAGE_BEST`, `STORAGE_MUTE`, `STORAGE_DIFF`,
   `STORAGE_REDUCED`, all prefixed `skyfighter.`.
2. **Tuning tables**: `DIFFICULTIES` (spawn rates, enemy speed, shoot chance, lives,
   bombs, boss HP multiplier), `KILLS_PER_STAGE`, `THEMES`, `ENEMY_TYPES`, `BOSSES`.
3. **Audio**: a Web Audio synth (`ensureAudio`, `beep`, `noiseBurst`). Sound effects are
   generated, not loaded from files.
4. **Input**: keyboard (`keys[e.code]`), a virtual joystick for touch, fire and bomb
   buttons, and mouse or touch on the canvas.
5. **Game state and update**: `resetGame`, `update(dt)`, `spawnEnemy`, `spawnBoss`,
   `maybeSpawnPowerUp`, `applyPowerUp`, `useBomb`, `fireWeapon`, `addScore`, `breakChain`.
6. **Boss patterns**: `BOSS_PATTERNS` and the `fire*` helpers (`fireRadial`,
   `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`, `fireCross`,
   `fireHomingSeeded`, `fireDoubleRing`), driven by `updateBoss`.
7. **Rendering**: `draw` and the `draw*` helpers (`drawPlane`, `drawBossShip`).
   Drawing is done with canvas 2D primitives only.
8. **Main loop**: `loop(now)` calls `trackPerf` and then `update` and `draw`, scheduled
   with `requestAnimationFrame`. Delta time is clamped to 50 ms.

## Conventions

- **Keep it one file.** The game is deliberately a single page with no external
  dependencies, scripts, or stylesheets. Do not add a CDN, a framework, or a build step
  without the user asking for it.
- **Plain JavaScript and CSS only.** Use `const`/`let`, arrow functions, and template
  literals as the existing code does. Do not add TypeScript or transpilation.
- **Match the existing style.** Two-space indentation, single quotes in JS, short
  helper functions, and `UPPER_SNAKE_CASE` for tuning constants and storage keys.
- **Tuning lives in constants.** Change balance through `DIFFICULTIES`, `BOSSES`,
  `ENEMY_TYPES`, and similar tables rather than hard-coding numbers in the update loop.
- **Respect the reduced-motion and quality settings.** Visual effects such as screen
  shake and particle counts are gated by `reducedMotion` and the adaptive `quality`
  level set in `trackPerf`. New effects should honor both.
- **Keep the game playable on touch and small screens.** The canvas is scaled through
  `resize()` against a `480 × 720` base size, and `touch-action: none` is set. Check
  new UI at phone width.
- **Persist only through the existing keys.** Use the `skyfighter.*` `localStorage`
  keys. Wrap new storage access so the game still runs if storage is unavailable
  (private windows, blocked site data).

## Working on the code

- **No automated tests exist.** Verify changes by loading `index.html` in a browser and
  playing through the affected path: start, fire, bomb, a power-up, a boss fight, game
  over, and restart. Also check mute, difficulty, and reduced-motion toggles if touched.
- **Check for JavaScript errors** in the browser console after any change to the script.
- **Keep the game loop fast.** Avoid allocating new arrays or objects every frame in
  `update` and `draw` when a reusable structure will do.
- **Do not break the DOM contract.** The script looks up elements by `id` at startup
  (`gameCanvas`, `overlay`, `overlayTitle`, `overlaySub`, and others). Renaming or
  removing an id requires updating the matching `getElementById` call.

## Git and delivery

- Default working branch for this session: `claude/gifted-tesla-r3e73f`. Commit there
  and push with `git push -u origin <branch>`.
- Write commit messages in the imperative mood, with a short subject and an optional
  body explaining the why.
- Do not open a pull request unless the user explicitly asks for one.
