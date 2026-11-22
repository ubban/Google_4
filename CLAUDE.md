# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo is **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs in the browser. It is a single self-contained file:

- `index.html` — all markup, CSS (`<style>`), and game logic (one inline `<script>` wrapped in an IIFE, `'use strict'`). About 1,670 lines.
- `README.md` — minimal placeholder.

There is no build step, no package manager, no bundler, no test suite, and no external dependencies. Do not add any of these unless the user asks.

## Running the game

Open `index.html` directly in a browser, or serve the directory with any static server (for example `python3 -m http.server`). No install is required.

## Code layout inside `index.html`

- **Styles** (approx. lines 10–360): dark theme via `color-scheme: dark`, HUD pills, overlay/menu, touch controls, vignette/flash effects, and `@keyframes`.
- **Markup** (approx. lines 360–420): a single `<canvas id="gameCanvas">`, HUD elements, overlay, difficulty buttons, touch controls (joystick, fire, bomb), and an `aria-live` region (`#srLive`).
- **Script** (approx. lines 422–end): one IIFE. Sections, roughly in order:
  - DOM references and `STORAGE_*` keys (`localStorage` keys are namespaced `skyfighter.*`).
  - `announce()` for screen-reader text, `trackPerf()`, motion/mute toggles.
  - Audio: lazily created `AudioContext` via `ensureAudio()`, with `beep()` and `noiseBurst()` helpers and a named `sfx` table.
  - Game data constants: `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES`, `ENEMY_TYPES`, `BOSSES`, `BOSS_PATTERNS`.
  - Input: keyboard (`keys[e.code]`), virtual joystick and buttons for touch, and mouse on the canvas.
  - Rendering: `drawPlane()`, `drawBossShip()`, and related draw functions.
  - Gameplay: `spawnEnemy()`, `spawnBoss()`, `maybeSpawnPowerUp()`, `applyPowerUp()`, `useBomb()`, `fireWeapon()`, `addScore()`/combo chain, `update(dt)`, and stage progression.
  - Lifecycle: `resetGame()`, game over, and best-score persistence.

When changing behavior, search for the relevant function or constant by name rather than relying on line numbers, which drift as the file changes.

## Conventions

- **Keep it a single file.** New game logic, styles, and markup belong in `index.html` unless the user asks for a split.
- **Preserve the IIFE and `'use strict'`.** Do not leak globals. Expose nothing on `window` unless necessary.
- **Fixed logical resolution.** The canvas is `BASE_W × BASE_H` (480 × 720) and is scaled by `resize()` for display. Use these logical coordinates in game code; do not key game logic to CSS pixels.
- **Time-based updates.** `update(dt)` takes a delta time. Scale movement and timers by `dt` so that speed does not depend on frame rate.
- **Data-driven tuning.** Put tunable numbers (difficulty multipliers, enemy stats, boss HP curve, spawn rates) in the existing constant tables (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `THEMES`) instead of scattering magic numbers.
- **Stage cycling.** Themes and bosses cycle with `% length`, so adding entries to `THEMES` or `BOSSES` extends the cycle automatically. Check that `resetGame()` and the stage-advance path still work after such changes.
- **Persistence.** Use the `skyfighter.*` `localStorage` keys. Wrap new storage reads and writes in the same defensive style used now (`Number(...) || default`, `=== '1'` for booleans).
- **Accessibility.** Keep `prefers-reduced-motion`/reduced-motion handling in mind for new screen shake, flashes, or animation (there is a `reducedMotion` setting). Announce important state changes through `announce()` so screen readers get them.
- **Touch and mouse parity.** Any new control must work on touch (`touchstart`/`touchend` with `preventDefault` where needed) and on desktop input.
- **Audio gating.** Start audio only from a user gesture via `ensureAudio()`. Respect the `muted` flag in all new sound effects.
- **Match the existing style.** Two-space indentation, single quotes in JavaScript, arrow functions for callbacks, and `const`/`let` (no `var`).

## Testing and verification

There is no automated test suite. To verify a change:

1. Load `index.html` in a browser (or a headless browser) and confirm there are no console errors on load.
2. Exercise the changed path: start a run, fire, use a bomb, collect power-ups, clear a stage boss, and reach game over.
3. Check the three difficulty settings if the change touches balance.
4. Check the layout at a phone-sized viewport and at desktop size, with the touch controls visible on touch-capable viewports.

If you cannot run the game in a browser, say so explicitly in your summary and describe what you checked by reading the code.

## Git and workflow

- Develop on the branch the environment specifies. Do not push to other branches without permission.
- Make focused commits with clear messages describing the gameplay or code change.
- Do not create pull requests unless the user asks for one.
- Do not commit secrets or local configuration files.
