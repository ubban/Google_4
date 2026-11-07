# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Google_4 is a small static web project containing **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs in the browser. The entire game lives in one file, `index.html` (about 1,670 lines): inline CSS in `<style>`, markup in `<body>`, and one inline IIFE in `<script>`. There is no build step, no bundler, no package manager, no test suite, and no external runtime dependencies.

Files:

- `index.html`: the complete game (markup, styles, and logic).
- `README.md`: a stub with a title only.
- `CLAUDE.md`: this file.

## Running and testing

- Open `index.html` directly in a browser, or serve the directory with any static server (for example `python3 -m http.server`).
- There are no automated tests, linters, or build commands. Verify changes by playing the game in a browser, on both desktop and a touch-sized viewport.

## Architecture of `index.html`

All game code sits inside a single `(function () { 'use strict'; ... })();` block. Sections, in file order:

1. **DOM references**: `const` lookups for every HUD element, overlay element, and button.
2. **Persistence keys**: `localStorage` keys prefixed `skyfighter.` (`bestScore`, `muted`, `difficulty`, `reducedMotion`).
3. **Accessibility and perf helpers**: `announce()` writes to the `#srLive` live region; `trackPerf()` samples frame times and steps a `quality` level (0 full, 1 medium, 2 low).
4. **Tuning tables**:
   - `DIFFICULTIES`: spawn rates, enemy speed and fire chance, starting lives and bombs, and boss HP multiplier for `easy`, `normal`, and `hard`.
   - `KILLS_PER_STAGE` (18): kills needed to advance a stage.
   - `THEMES`: six sky palettes that cycle by stage.
   - `ENEMY_TYPES`: `grunt`, `interceptor` (weaving), and `tank` (4 HP, unlocks at stage 2).
   - `BOSSES`: six bosses (VANGUARD through TITAN) with colors and HP multipliers.
5. **Audio**: a lazily created `AudioContext` (`ensureAudio()`), with `beep()` and `noiseBurst()` primitives and a named `sfx` object. Audio starts only after a user gesture.
6. **Effects and HUD**: `renderHearts`, `renderBombs`, `flashHit`, `flashBomb`, `showComboPop`, `showStageBanner`, `screenShake`.
7. **Layout**: `BASE_W = 480`, `BASE_H = 720`. `resize()` scales the canvas to fit the window, capped at 1.6x. The canvas backing store is always 480x720 in game coordinates.
8. **Input**: keyboard (`keys` map keyed by `e.code`), touch joystick plus FIRE and BOMB buttons, and mouse steering with click-to-fire on the canvas. Keyboard and joystick input take priority over mouse steering.
9. **Game state**: globals declared at the top of the game-state section (`player`, `bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`, `boss`, plus score, lives, bombs, chain, and multiplier). `resetGame()` reinitializes them.
10. **Simulation**: `update(dt)` runs movement, firing, spawning, collisions, and stage progression. Helpers include `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`, `addScore`, `breakChain`, `useBomb`, `fireWeapon`, `applyPowerUp`, `updateBoss`, `killBoss`, `loseLife`, and `endGame`.
11. **Boss attacks**: a set of pattern functions (`fireRadial`, `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`, `fireCross`, `fireHomingSeeded`, `fireDoubleRing`) combined in `BOSS_PATTERNS`.
12. **Rendering**: `draw()` renders everything to the canvas with `drawPlane`, `drawBossShip`, and `drawClouds`. Screen shake is applied during rendering.
13. **Main loop**: `loop(now)` clamps `dt` to 50 ms, updates and draws only while `running && !paused`, and calls `requestAnimationFrame`. Game start and resume are wired to `startBtn`.

Key game rules:

- **Weapons**: `vulcan` (spread shots, up to 3 bullets at level 3) and `laser` (piercing, up to level 3). Power-up `weapon` raises the level from 1 to 3, then switches weapon type and resets to level 1.
- **Power-ups**: `life` (capped at max lives), `bomb` (capped at max bombs), and `weapon`.
- **Scoring**: `addScore()` increments `chain`, and the multiplier is `1 + min(9, floor(chain / 5))` (up to x10). The chain resets on a timed-out chain window (2.2 s) or on losing a life.
- **Game over**: `endGame()` saves a new best to `localStorage` only when the score beats the stored best.

## Conventions

- Keep the single-file structure unless the user asks for a split. Do not introduce a build step, framework, or package manager without asking.
- Put new tuning values in the existing constant tables (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `THEMES`) rather than hardcoding them in logic.
- Use `const`/`let`, arrow functions, and the existing naming style (camelCase for functions and variables, UPPER_SNAKE_CASE for constant tables).
- Keep all new game state inside the IIFE. Do not add globals to `window`.
- Respect the reduced-motion setting: new shake, flash, or pulse effects should check `reducedMotion` (as `screenShake()`, `flashHit()`, and `flashBomb()` already do).
- Keep the mobile controls working. Any new input must also have a touch path, and any new HUD element must fit the narrow-viewport layout (`clamp()` sizing is already used).
- Preserve the accessibility hooks: announce important state changes through `announce()` and keep the `#srLive` region intact.
- Persistent data uses the `skyfighter.` key prefix. Wrap new `localStorage` access in a way that tolerates failures (private mode, blocked storage).

## Git workflow

- Develop on the branch assigned to the session (currently `claude/gifted-tesla-bxfpe9`). Do not push to other branches without explicit permission.
- Write clear, descriptive commit messages in the imperative mood.
- Do not open a pull request unless the user explicitly asks for one.
