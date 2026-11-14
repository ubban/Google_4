# CLAUDE.md

Guidance for Claude Code when working in this repository.

## What this repo is

`Google_4` is a small, dependency-free static web project. Its only game is **Sky Fighter**, a Raiden-style vertical shoot-'em-up rendered on a `<canvas>`. There is no build step, package manager, test suite, or server code.

## Layout

| Path        | Purpose |
|-------------|---------|
| `index.html` | The entire game: markup, inline `<style>`, and one inline `<script>` IIFE (~1,670 lines). |
| `README.md`  | Minimal project readme (title only). |

Open `index.html` directly in a browser to run the game. Nothing needs to be installed.

## How `index.html` is organized

- **`<style>` (lines ~10–361):** all CSS, including the HUD, overlay/menus, touch controls and light/dark theming.
- **Body markup (~363–421):** HUD elements (score, best, multiplier, weapon, hearts, bombs, boss bar), overlay menus, touch controls, and a single `<canvas id="gameCanvas">`.
- **`<script>` (~422–1668):** one `'use strict'` IIFE. Main sections, in order:
  - DOM references and constants: `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES`, `ENEMY_TYPES`, `BOSSES`, and `STORAGE_*` localStorage keys (`skyfighter.*`).
  - Audio: `ensureAudio`, `beep`, `noiseBurst` (Web Audio, synthesized; no audio files).
  - HUD and effects: `renderHearts`, `renderBombs`, `showComboPop`, `showStageBanner`, `screenShake`.
  - Input: `keydown`/`keyup` handlers, pointer and touch joystick, fire and bomb buttons, `resize` (fixed `BASE_W × BASE_H = 480 × 720` logical resolution).
  - Game state: `resetGame`, `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`, `applyPowerUp`, `useBomb`, `fireWeapon`.
  - Boss attacks: `BOSS_PATTERNS` and the `fire*` pattern helpers (radial, spread, columns, spiral, cross, homing, double ring).
  - Per-frame: `update(dt)`, `draw()`, `drawPlane`, `drawBossShip`, `drawClouds`.
  - Loop: `loop(now)` runs on `requestAnimationFrame` and clamps `dt` to 50 ms. `trackPerf` adapts a `quality` level (0–2) from the average frame time.

## Controls (for reference when changing input code)

- Move: Arrow keys or WASD. Fire: Space (or hold the on-screen fire button). Bomb: X or Shift.
- Pause: P or Escape.
- Touch devices get a joystick, fire and bomb buttons (`isTouchDevice` check).

## Conventions

- Keep the game in the single file unless the user asks to split it. Do not introduce a build step, bundler, or npm dependencies without being asked.
- Keep the script wrapped in its IIFE and `'use strict'`. Do not leak globals.
- Use `const`/`let` and the existing naming style (camelCase functions, UPPER_SNAKE constants).
- Persistent settings go through `localStorage` under the `skyfighter.*` keys. Wrap reads and writes in `try/catch`, since storage can be blocked (for example in private windows).
- Respect the existing accessibility hooks: `prefers-reduced-motion` handling (`reducedMotion`, `screenShake`), the `srLive` live region used by `announce()`, and the `aria` attributes on controls.
- Keep the game playable at phone widths. Preserve the `viewport` meta and the `resize()` logic.
- Frame-rate-dependent code should use the `dt` argument, not fixed per-frame steps, unless the surrounding code already does so.

## Working in this repo

- There are no automated tests or linters configured. Verify changes by loading `index.html` in a browser (for example with the Playwright setup available in this environment) and playing through the affected behavior. Check the browser console for errors.
- Before editing, read the relevant section of `index.html` with line ranges, since the file is large.
- Keep commits focused. Only commit or push when asked, and push only to the branch the session specifies.
