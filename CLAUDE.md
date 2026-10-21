# CLAUDE.md

Guidance for Claude Code when working in this repository.

## What this repo is

A small, dependency-free browser game: **Sky Fighter**, a Raiden-style vertical shoot-em-up.
The entire game lives in a single file, `index.html` (~1,670 lines): markup, CSS, and one inline
`<script>` block wrapped in an IIFE. `README.md` is a placeholder.

There is no build step, package manager, test suite, or linter configured. To run the game, open
`index.html` in a browser (or serve the directory with any static server).

## Layout of `index.html`

- **Lines ~1–360 — `<style>`.** Layout, HUD, overlay, touch controls, and `.reduced-motion` styles.
- **Lines ~362–420 — markup.** `#container` holds the `#gameCanvas`, HUD (`#hud`, `#bossBarWrap`,
  `#topBar`), touch controls (`#touchControls`), and the start/pause overlay (`#overlay`).
- **Lines ~422–1668 — `<script>`.** One IIFE with `'use strict'`. Sections, in order:
  1. DOM references and `STORAGE_*` keys
  2. Adaptive performance (`trackPerf` lowers quality when frame time is high)
  3. Config tables: `DIFFICULTIES`, `THEMES`, `ENEMY_TYPES`, `BOSSES`, `KILLS_PER_STAGE`
  4. Audio (`ensureAudio`, `beep`, `noiseBurst`, the `sfx` helpers)
  5. HUD and feedback helpers (hearts, bombs, combo pop, stage banner, screen shake)
  6. Input: keyboard, joystick, touch buttons, and mouse steering on the canvas
  7. Game state: `resetGame`, `update(dt)`, `draw()`, `spawnEnemy`, `spawnBoss`, power-ups,
     `addScore`/combo chain, `useBomb`, `fireWeapon`
  8. Boss patterns: `BOSS_PATTERNS` and the `fire*` helpers, `updateBoss`, `killBoss`
  9. Lifecycle: `loseLife`, `endGame`, `togglePause`
  10. Rendering: `drawPlane`, `drawBossShip`, `drawClouds`, `draw()`
  11. `loop(now)` (the `requestAnimationFrame` driver) and the start button handler

Keep new code in the matching section rather than adding a new top-level block.

## Game facts worth knowing before editing

- **Logical resolution:** 480×720 (`BASE_W`/`BASE_H`). The canvas is scaled to fit the window
  (capped at 1.6×), so all game coordinates are in this space.
- **Frame step:** `dt` is clamped to 50 ms in `loop`. Movement and timers should use `dt`, not frame counts.
- **Difficulty** (`DIFFICULTIES`) sets spawn rate, enemy speed, enemy fire chance, starting lives,
  starting bombs, and boss HP multiplier. The choice persists in `localStorage`.
- **Stages:** `KILLS_PER_STAGE` (18) kills advance the stage. Each stage cycles through `THEMES`
  for the background palette. Bosses come from `BOSSES`, and each has its own attack pattern.
- **Enemies:** `grunt`, `interceptor` (weaves), and `tank` (4 HP, unlocks at stage 2). The
  `minStage` field gates spawning.
- **Power-ups:** `life`, `bomb`, and `weapon` (toggles between `vulcan` and `laser`).
- **Persistence:** `localStorage` keys are `skyfighter.bestScore`, `skyfighter.muted`,
  `skyfighter.difficulty`, and `skyfighter.reducedMotion`. Reduced motion defaults to the
  `prefers-reduced-motion` media query until the player toggles it.
- **Accessibility:** announcements go through the `#srLive` `aria-live` region via `announce()`.
  Keep this in mind when adding new player-facing messages.
- **Controls** are listed in the `#controlsHint` text. If you change a binding, update that text too.
  Current bindings: arrows/WASD move, Space fires, X or Shift bombs, P or Esc pauses, mouse
  steers and clicks to fire.

## Conventions

- Match the existing style: 2-space indentation, single quotes, `const`/`let`, arrow callbacks,
  and `// ---------- Section ----------` dividers between sections.
- Keep it a single self-contained file with no external dependencies unless the user asks otherwise.
- Don't introduce a framework or build step for small changes.
- Prefer tuning values in the config tables over hard-coding numbers in update/draw code.
- Keep the mobile path working: touch controls are shown only when `isTouchDevice` is true.

## Verifying changes

There are no automated tests. After editing:

1. Open `index.html` in a browser (or a headless browser) and check the console for errors.
2. Play through: start, fire, bomb, take damage, pause/resume, reach a boss, and game over.
3. Check that the best score, mute, difficulty, and reduced-motion settings persist across reloads.
4. Resize the window and, if relevant, check touch controls with device emulation.

## Git

- Development branch for this work: `claude/gifted-tesla-8tc63s`. Push there with `git push -u origin <branch>`.
- Don't open a pull request unless the user asks for one.
