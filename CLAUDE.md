# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a small static web project. Its only real content is **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs in the browser as a single HTML5 canvas page.

- `index.html` (~1,670 lines): the entire game — inline CSS, markup, and one IIFE `<script>` with all game logic, rendering, audio, and input.
- `README.md`: a one-line title. Nothing else documents the project yet.

There is no build step, no package manager, no framework, no test suite, and no CI config. Do not add any of these unless asked.

## Running and testing

- Open `index.html` directly in a browser, or serve the folder with any static server (e.g. `python3 -m http.server`) if a module/CORS-sensitive feature is ever added.
- There are no automated tests. Verify changes by loading the page and playing: start a run, fire, use a bomb, pause/resume, and reach a boss, on both desktop and a touch-width viewport.
- Check the browser console for errors after every change to the script.

## Code layout inside `index.html`

- **`<style>`**: HUD pills, overlay card, vignette/flash effects, touch controls, keyframes. Layout must work at phone width (no horizontal scroll, `touch-action: none` on the canvas and body is intentional).
- **`<body>`**: `#container` wraps `#gameCanvas` plus HUD, overlay, touch-control, and boss-bar elements. Element IDs are referenced by `getElementById` at the top of the script — renaming an ID requires updating that list.
- **`<script>`** (single IIFE, `'use strict'`), roughly in this order:
  1. DOM references and `STORAGE_*` keys.
  2. `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES` (per-stage sky palettes), and the boss design table (`VANGUARD`, `BEHEMOTH`, `SPECTER`, `DREADNOUGHT`, `WRAITH`, `TITAN`, each with `hpMul`).
  3. Audio: `ensureAudio`, `beep`, `noiseBurst` (Web Audio, created lazily on first user gesture).
  4. HUD helpers: `renderHearts`, `renderBombs`, `showComboPop`, `showStageBanner`, `screenShake`.
  5. Input: `keys` map, keyboard handlers, touch joystick (`joystick*`), `canvasPointFromEvent`.
  6. Game state: `resetGame`, `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`, `spawnExplosion`, `addScore` / `breakChain` (kill-chain multiplier), `useBomb`, `fireWeapon`.
  7. Boss attack patterns: `fireRadial`, `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`, `fireCross`, `fireHomingSeeded`, `fireDoubleRing`, dispatched from `updateBoss`.
  8. Main loop: `update(dt)` (logic), `draw()` (rendering, including `drawPlane`, `drawBossShip`, `drawClouds`), `loop(now)` (requestAnimationFrame driver; `dt` is wall-clock time clamped to 50 ms).
  9. Run lifecycle and setup: `togglePause`, `endGame`, `setDifficulty`, `resize`, `trackPerf`.

## Game rules and constants (quick reference)

- **Difficulty** (`easy` / `normal` / `hard`) controls spawn rate, enemy speed, enemy fire chance, starting lives, starting bombs, and boss HP multiplier. Selection persists in `localStorage`.
- **Stages**: a stage ends after `KILLS_PER_STAGE` (18) kills, then a boss fight starts. Stage N uses `BOSSES[(N-1) % 6]` and `THEMES[(N-1) % 6]`, so both cycle every six stages.
- **Weapons**: Vulcan and Laser, each with 3 power levels. Power-ups raise weapon level, add a bomb, or grant an extra life.
- **Bombs** clear the screen and damage bosses. Bomb count is limited per difficulty.
- **Score**: kill-chain multiplier, reset by `breakChain`. Best score persists.

### Controls

| Action | Keyboard | Touch |
| --- | --- | --- |
| Move | Arrow keys or `W`/`A`/`S`/`D` | Joystick (left zone) |
| Fire | `Space` (hold) | Fire button |
| Bomb | `X` or `Shift` | Bomb button |
| Pause | `P` or `Esc` | Pause button |
| Mute | Mute button | Mute button |

## localStorage keys

All persisted state uses these keys. Keep names stable, since changing them silently resets players' data.

- `skyfighter.bestScore`
- `skyfighter.muted`
- `skyfighter.difficulty`
- `skyfighter.reducedMotion`

Every `localStorage` access should stay wrapped so the game still works when storage is unavailable (private mode, blocked site data). Check that the existing reads/writes still behave this way if you touch them.

## Conventions

- Keep everything in `index.html` unless there is a clear reason to split files. Do not introduce a bundler, framework, or npm dependencies for this project.
- Stay in the existing style: vanilla JS, `const`/`let`, function declarations inside the IIFE, no globals added to `window`.
- Game-loop code should use the `dt` passed into `update`/`updateBoss` rather than reading the clock directly. `dt` is already clamped, so keep new timers in seconds and scale them by `dt`.
- Respect the reduced-motion setting when adding visual effects (screen shake, flashes, pulses). Gate new motion behind the same flag used by existing effects.
- Keep touch and keyboard controls in parity: any new player action needs both an input path and a visible touch control.
- Accessibility: the page has an `srLive` live region used by `announce(text)`. Use it for important state changes (stage start, boss warning, game over).

## Git workflow

- Develop on the branch specified by the task (currently `claude/gifted-tesla-rr0w69`). Do not push to other branches without explicit permission.
- Write clear, descriptive commit messages in the imperative mood.
- Do not open a pull request unless explicitly asked.

## Things to avoid

- Do not add a build system, package.json, or external CDN dependencies without a request.
- Do not rename `STORAGE_*` keys or change the meaning of stored values.
- Do not remove the `touch-action: none` / `user-select: none` rules; they prevent gesture and selection interference during play.
