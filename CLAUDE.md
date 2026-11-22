# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo is a small single-page game, **Sky Fighter**: a Raiden-style vertical shoot-em-up built as one self-contained HTML5 canvas app.

- `index.html` (~1,670 lines): the entire game. Markup, CSS (`<style>` block, ~lines 10–360), and JavaScript (`<script>` block starting ~line 422, wrapped in an IIFE with `'use strict'`).
- `README.md`: placeholder. Its content is a stray timestamp line, not documentation.

There is no build step, bundler, package manager, framework, or test suite. Do not add one unless asked. To run the game, open `index.html` in a browser.

## Repository layout and conventions

- Keep the game as a single file unless the user asks to split it. Do not introduce external runtime dependencies or CDN scripts. The game currently needs none.
- Audio is generated with the Web Audio API (`ensureAudio`, `beep`, `noiseBurst`). Do not add audio asset files without asking.
- Rendering is on a `<canvas id="gameCanvas">` using the 2D context. The logical play field is `BASE_W = 480` by `BASE_H = 720`, scaled by `resize()`. Keep gameplay coordinates in that logical space.
- The HUD, overlay, and touch controls are DOM elements positioned over the canvas. Styles live in the `<style>` block using CSS custom properties and existing class names. Reuse them before adding new ones.

## Code structure inside the script

The script is organized by section comments and top-level `const` tables. Find things by these names:

- **Tuning tables:** `DIFFICULTIES` (spawn rates, enemy speed, shot chance, lives, bombs, boss HP multiplier), `KILLS_PER_STAGE`, `THEMES` (per-stage sky palettes), `ENEMY_TYPES`, `BOSSES`, `BOSS_PATTERNS`.
- **Persistence:** `localStorage` keys are prefixed `skyfighter.` (`bestScore`, `muted`, `difficulty`, `reducedMotion`). Reads and writes should stay tolerant of missing or invalid values.
- **Game state and flow:** `resetGame`, `update(dt)`, `draw()`, `loop(now)`, `endGame`, `togglePause`, `loseLife`.
- **Combat:** `spawnEnemy`, `spawnBoss`, `updateBoss`, `killBoss`, `fireWeapon`, `useBomb`, `applyPowerUp`, and the `fire*` pattern helpers used by bosses.
- **Scoring:** `addScore` and `breakChain` implement the kill-chain multiplier.
- **Input:** keyboard (`keydown` on `window`), mouse/pointer steering and firing via `canvasPointFromEvent`, and touch joystick/buttons (`joystickStart/Move/End`). Preserve all three input paths when changing controls.
- **Accessibility and comfort:** `announce()` drives the `aria-live` region. Reduced-motion mode disables screen shake, flash, and pulse effects. Any new visual effect that shakes, flashes, or pulses must respect that toggle.
- **Adaptive quality:** `trackPerf` lowers quality when frames run slow. Keep expensive draw work compatible with it.

## Working on the game

- **Frame timing:** `dt` is clamped to 50 ms in `loop`. Make all motion and timers `dt`-based, not frame-based.
- **Randomness:** gameplay uses `Math.random()`. Do not add seeded or deterministic logic unless asked.
- **Style:** match the existing code. Use `const`/`let`, arrow functions for callbacks, two-space indentation, single quotes, and terse one-line helpers where the file already does that.
- **Scope:** keep changes minimal and local. Do not refactor the IIFE structure, rename shared tables, or reorganize the file as a side effect of a feature or fix.

## Verifying changes

No automated tests exist. Verify by:

1. Opening `index.html` in a browser (or a headless browser for automated checks).
2. Starting a game, then checking the affected path: movement, firing, bombs, a stage transition, boss fight, pause, game over, and the best-score update.
3. Checking the browser console for errors.
4. Checking mobile layout and touch controls when the change touches the HUD, overlay, or input.

If a change cannot be verified this way, say so in the final message rather than claiming it works.

## Git

- Develop on the branch assigned for the session. Do not push to other branches without permission.
- Keep commit messages descriptive, in the imperative mood, with a summary line followed by a short body when the change needs context.
