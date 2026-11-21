# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a small static web project. Its only real content is **Sky Fighter**, a Raiden-style vertical shoot-'em-up written as a single-page HTML5 canvas game.

- `index.html` (~1,670 lines): the entire game — markup, CSS, and JavaScript — in one file. No build step, no dependencies, no package manager.
- `README.md`: a one-line title stub.

There is no test suite, linter, or CI configuration in the repo.

## Running the game

No build is required. Open `index.html` directly in a browser, or serve the directory with any static server, e.g.:

```bash
python3 -m http.server 8000   # then visit http://localhost:8000/index.html
```

## Code structure of `index.html`

- **`<style>`** (around lines 10–361): all CSS for the HUD, overlays, touch controls, and animations.
- **`<body>`** (around line 363+): HUD elements, overlay/start card, pause and mute/motion buttons, touch joystick and fire/bomb buttons, and `<canvas id="gameCanvas">`.
- **`<script>`** (around line 422+): one IIFE with `'use strict'`. Everything lives inside it; nothing is exported to `window`.
  - **Config tables** near the top: `DIFFICULTIES` (spawn rates, enemy speed/fire chance, lives, bombs, boss HP multiplier), `KILLS_PER_STAGE`, `THEMES` (per-stage sky palettes), `ENEMY_TYPES`, `BOSSES`.
  - **Persistence**: `localStorage` keys prefixed `skyfighter.` (`bestScore`, `muted`, `difficulty`, `reducedMotion`). Reads and writes are direct; keep them that way unless adding a wrapper for all of them.
  - **Audio**: Web Audio API, created lazily by `ensureAudio()` on first user gesture (required by browser autoplay policy). Sound effects are synthesized via `beep()` and `noiseBurst()`.
  - **Input**: keyboard (`keys[e.code]`), mouse on canvas, and touch joystick + buttons. `P` or `Escape` toggles pause.
  - **Game loop**: `update(dt)` advances state, the `drawX` functions render, and `loop` is driven by `requestAnimationFrame`. Entities are plain objects in arrays.
  - **Weapons and bombs**: `fireWeapon()` (Vulcan/Laser, 3 power levels), `useBomb()` (clears the screen, damages bosses).
  - **Bosses**: `spawnBoss()`, `updateBoss()`, and the `BOSS_PATTERNS` array of bullet patterns (radial, aimed spread, columns, spiral, cross, homing, double ring).
  - **Responsive scaling**: `resize()` fits a fixed `BASE_W × BASE_H` (480×720) logical canvas to the viewport.

## Conventions

- Keep the game as a single self-contained HTML file unless the user asks to split it. Do not add a bundler, framework, or npm dependencies without being asked.
- Stay in the existing style: `const` by default, `camelCase` for functions and variables, `UPPER_SNAKE_CASE` for module-level config constants, two-space indentation.
- Put new tunable numbers (enemy speeds, spawn rates, boss HP, scoring) in the existing config tables rather than inline magic numbers.
- Any new persisted setting should use the `skyfighter.` key prefix and follow the existing load-at-start / save-on-change pattern.
- Preserve accessibility hooks: the `srLive` live region is used for announcements via `announce()`, and the reduced-motion setting (`reducedMotion`, which also honors `prefers-reduced-motion`) dampens screen shake and hit flashes. Keep that behavior when adding new motion effects.
- Keep the mobile layout working. The viewport meta disables user scaling, and touch controls are shown for touch devices; check both when changing HUD or overlay layout.

## Verifying changes

With no tests in the repo, verify by running the game:

1. Serve the directory (see above) and load `index.html`.
2. Check the start overlay, a full play session, pause (`P`/`Escape`), game over, and the high-score update.
3. Exercise difficulty switching, mute, and the reduced-motion toggle.
4. For layout changes, check at a phone-sized viewport as well as desktop.
5. Watch the browser console for errors; the game should run with none.

If you have Playwright available, a headless smoke test that loads the page and checks for console errors is a cheap way to catch regressions.

## Git

- Default development branch for this session: `claude/gifted-tesla-ge2806`.
- Commit messages should be short and imperative, with a body explaining what changed and why when it isn't obvious from the diff.
- Do not create pull requests unless explicitly asked.
