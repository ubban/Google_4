# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a small static project. Its only real content is **Sky Fighter**, a
Raiden-style vertical shoot-'em-up written as a single-page HTML5 canvas game.

Files:

- `index.html` (~1,670 lines): the entire game. Inline `<style>` (lines ~10–360),
  the HUD/overlay/touch-control markup (lines ~361–421), and one inline `<script>`
  (lines ~422–end) wrapped in an IIFE with `'use strict'`.
- `README.md`: a one-line placeholder. Its content is not meaningful documentation.

There is no build step, no package manager, no test suite, and no dependencies.
Audio is synthesized with the Web Audio API; nothing is fetched from the network.

## Running and verifying

- Open `index.html` directly in a browser, or serve the directory with any static
  server (for example `python3 -m http.server`).
- Verify changes by playing in a browser. There are no automated tests or linters.
- For touch behavior, use the browser device emulator. Touch controls appear on
  small viewports.

## Code architecture (`index.html`)

All game logic lives in the IIFE. Key sections, in source order:

1. **DOM lookups**: every element is fetched once by id at the top of the script.
   Element ids are part of the contract with the markup and CSS. Renaming an id
   breaks the game silently, so update the markup and script together.
2. **Persistence**: `localStorage` keys are namespaced `skyfighter.*`
   (`bestScore`, `muted`, `difficulty`, `reducedMotion`). Every access must
   tolerate missing storage, so keep reads defaulted.
3. **Accessibility helpers**: `announce()` writes to the `srLive` live region.
   Reduced motion is toggled through a body class (`reduced-motion`) and the
   `prefers-reduced-motion` media query.
4. **Adaptive quality**: `trackPerf()` averages frame times over 45 frames and steps
   a `quality` level (0 full, 1 medium, 2 low) to shed effects on slow devices.
5. **Config tables** (the main tuning knobs):
   - `DIFFICULTIES`: spawn rate, enemy speed, shoot chance, starting lives and
     bombs, and boss HP multiplier for easy, normal, and hard.
   - `KILLS_PER_STAGE` (18): kills needed before the stage boss is triggered.
     Defeating the boss advances the stage.
   - `THEMES`: six sky palettes, selected by `(stage - 1) % THEMES.length`.
   - `ENEMY_TYPES`: grunt, interceptor (weaving), and tank (4 HP, minimum stage 2).
   - `BOSSES`: six bosses with colors and an `hpMul`. Each boss has its own
     attack patterns in `BOSS_PATTERNS`.
6. **Audio**: `ensureAudio()` lazily creates the `AudioContext` on the first user
   gesture. `beep()` and `noiseBurst()` build sound effects, and the `sfx` object
   groups them. `muted` is honored by these helpers.
7. **Input**: keyboard (`keys[e.code]`), a virtual joystick and fire/bomb buttons
   for touch, and mouse aiming on the canvas. Input only sets flags such as
   `firing`, `keys`, and `mouseActive`. The `update()` loop reads them.
8. **Game state**: `resetGame()` initializes all state. Key variables include
   `score`, `lives`, `maxLives`, `running`, `paused`, `gameOver`, plus arrays for
   enemies, bullets, power-ups, and explosions.
9. **Update loop**: `update(dt)` advances the simulation by the elapsed time `dt`.
   Its responsibilities are: spawning, weapon firing, collision checks
   (`rectsOverlap`), kill-chain scoring (`addScore`, `breakChain`), power-ups
   (`applyPowerUp`), and bosses (`updateBoss`, `killBoss`).
10. **Weapons and bosses**: `fireWeapon()` handles Vulcan and Laser with three
    power levels. Boss attacks come from the radial, spread, column, spiral,
    cross, homing, and double-ring helpers referenced by `BOSS_PATTERNS`.
11. **Lifecycle**: `loseLife()`, `endGame()`, and `togglePause()`.
12. **Rendering**: `draw()` runs after `update()`. It draws parallax clouds
    (`drawClouds`), planes (`drawPlane`), bosses (`drawBossShip`), and HUD
    effects, such as the low-health vignette and hit flash.
13. **Main loop**: `loop(now)` drives `update` and `draw` with
    `requestAnimationFrame`. `resize()` fits a fixed `480×720` logical canvas
    (`BASE_W`, `BASE_H`) to the viewport, scaling up to at most 1.6×.

## Conventions

- Keep the single-file structure unless the user asks to split it. Adding a build
  step or a dependency is a significant change and needs the user's approval.
- Add new tuning values to the existing config tables (`DIFFICULTIES`,
  `ENEMY_TYPES`, `BOSSES`, `THEMES`) rather than hard-coding them in functions.
- Gameplay code must not assume `localStorage` or `AudioContext` is available.
  Guard both.
- Keep the HTML comments and section banners (`// ---------- Name ----------`)
  consistent with the existing style.
- Do not add external scripts, fonts, or CDN links without asking. The game is
  intentionally self-contained.

## Git

- Development branch for this repository: `claude/gifted-tesla-bbaref`.
- The repository history is short: an `Initial commit` followed by the Sky Fighter
  commit. Write commit messages in the same imperative, descriptive style.
