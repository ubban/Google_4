# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a small, dependency-free static web project. Its only real content is **Sky Fighter**, a Raiden-style vertical shoot-'em-up rendered on an HTML5 `<canvas>`.

Files:

- `index.html` (~1670 lines): the entire game. CSS in one `<style>` block (starts ~line 10), markup in the body, and all JavaScript in a single IIFE `<script>` (starts ~line 422).
- `README.md`: placeholder (title only).

There is no `package.json`, build step, bundler, linter, or test suite in the repo. Do not add tooling unless asked.

## Running and verifying

- Open `index.html` directly in a browser, or serve the directory with any static server (e.g. `python3 -m http.server`).
- Verification for gameplay changes has been done headlessly with Playwright. The pre-installed Chromium works via `require('playwright').chromium.launch()` with no options. Do not run `playwright install`. Check that the page logs no console errors, and exercise the code path you changed (firing, collisions, boss patterns, stage advance, pause, etc.).
- The game has no automated tests; manual or headless checks are the only verification.

## Architecture of `index.html`

Everything lives inside one `(function () { 'use strict'; ... })()` closure. Nothing is exported to `window`, so keep new code inside the closure.

Major sections, roughly in source order:

1. **DOM refs and storage keys**: element lookups by id; `localStorage` keys `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion`. Every storage read/write should tolerate being unavailable.
2. **Accessibility**: `announce()` writes to the `#srLive` aria-live region. Reduced motion defaults to `prefers-reduced-motion` and is toggled by `#motionBtn`; it dampens screen shake and disables flash/pulse animations.
3. **Adaptive performance**: `trackPerf()` keeps a 45-sample rolling frame-time average and steps `quality` between 0 (full), 1 (medium), and 2 (low), which trims particles and shadow-blur glow.
4. **Config tables**:
   - `DIFFICULTIES` (easy/normal/hard): spawn timing, enemy speed, enemy shoot chance, starting lives, starting bombs, boss HP multiplier.
   - `KILLS_PER_STAGE` (18): kills needed to trigger a boss.
   - `THEMES`: six background palettes, cycled by stage.
   - `ENEMY_TYPES`: `grunt`, `interceptor` (weaves), `tank` (4 HP, `minStage: 2`).
   - `BOSSES`: six designs (Vanguard, Behemoth, Specter, Dreadnought, Wraith, Titan), cycled by stage, each with its own `hpMul`.
   - `BOSS_PATTERNS` (~line 1372): attack patterns per boss, built from helpers such as `fireRadial`, `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`, `fireCross`, `fireHomingSeeded`, `fireDoubleRing`. Bosses telegraph each attack before firing.
5. **Audio**: a lazily created Web Audio synth (`ensureAudio`, `beep`, `noiseBurst`, and a `sfx` object). Master gain goes through a compressor. Sounds are short layered sequences with stereo pan from on-screen x (`panFor`). Audio starts only after a user gesture; call `ensureAudio()` from input handlers.
6. **HUD/effects**: `renderHearts`, `renderBombs`, `flashHit`, `flashBomb`, `showComboPop`, `showStageBanner`, `screenShake`.
7. **Layout**: `resize()` fits a fixed `BASE_W x BASE_H` (480 x 720) logical canvas into the viewport.
8. **Input**: keyboard (arrows/WASD move, Space or hold to fire, X/Shift bomb, P/Esc pause), mouse (move to steer, button to fire, crosshair), and touch (joystick in `#moveZone`, `#fireBtn`, `#bombBtn`). Keyboard and joystick take priority over mouse steering.
9. **Game state and logic**: `resetGame`, `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`, `addScore` (kill-chain multiplier, `breakChain`), `useBomb`, `fireWeapon` (Vulcan/Laser, 3 levels), `update(dt)`, `applyPowerUp`, `updateBoss`, `killBoss`, `loseLife`, `endGame`, `togglePause`.
10. **Rendering**: `drawPlane`, `drawBossShip`, `drawClouds`, `draw()`.
11. **Main loop**: `loop(now)` drives `update` and `draw` via `requestAnimationFrame`. `startBtn` starts a run.

## Conventions

- Plain ES2017+ JavaScript and CSS, no frameworks, no modules, no external network requests. Keep the single-file structure unless the user asks for a split.
- Use `const`/`let`, arrow functions, and the existing naming style (camelCase functions, UPPER_SNAKE for config constants).
- Add new tuning values to the config tables (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `THEMES`) rather than hard-coding magic numbers in logic.
- Keep the game usable on phones: preserve the `viewport-fit=cover` meta, safe-area insets, and touch controls when changing layout or input.
- Any new user-facing state change should be announced through `announce()` if it matters to screen-reader users, and any new animation should respect `reducedMotion`.
- Keep the `z-index` ordering in mind: the top bar (mute/motion/pause) must stay above the start overlay so it is clickable before a run begins.

## Git

- Development branch for this session: `claude/gifted-tesla-u4ezma`. Do not push to other branches without explicit permission.
- Commit messages follow the existing style: a short imperative subject, then a wrapped body explaining the why and how it was verified.
- Do not open pull requests unless explicitly asked.
