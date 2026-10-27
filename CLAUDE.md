# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Sky Fighter is a Raiden-style vertical shoot-'em-up that runs in the browser. The whole game is a single self-contained file, `index.html`: markup, CSS, and one inline `<script>` that draws everything on a `<canvas>`. `README.md` is a placeholder.

There is no build step, no package manager, no bundler, and no test suite. Do not add one unless asked.

## Running it

Open `index.html` directly in a browser, or serve the directory with any static server, for example `python3 -m http.server`. Nothing needs to be installed.

## Code layout (`index.html`)

- **Lines ~1–360, `<style>`**: dark theme (`color-scheme: dark`), HUD pills, overlay, touch controls, vignette and flash effects, and the `.reduced-motion` class.
- **Lines ~360–421, markup**: `#gameCanvas`, `#hud`, `#overlay`, `#touchControls` (joystick, fire, bomb), and buttons for mute, motion, pause, and difficulty. `#srLive` is an ARIA live region used by `announce()`.
- **Lines ~422–1669, one IIFE `<script>`** (`'use strict'`), with these sections in order:
  - DOM lookups and `STORAGE_*` keys.
  - Adaptive quality (`trackPerf`): averages the last 45 frame times and steps `quality` between 0 (full), 1 (medium), and 2 (low).
  - Tuning tables: `DIFFICULTIES`, `KILLS_PER_STAGE` (18), `THEMES` (six stage skies), `ENEMY_TYPES` (grunt, interceptor, tank), and `BOSSES` (six bosses).
  - Audio built on the Web Audio API (`ensureAudio`, `beep`, `noiseBurst`).
  - Input: keyboard, pointer on canvas, and touch joystick and buttons.
  - Game state and logic: `resetGame`, `spawnEnemy`, `spawnBoss`, `update(dt)`, `applyPowerUp`, `useBomb`, `fireWeapon`, and the boss attack patterns in `BOSS_PATTERNS` built from `fire*` helpers.
  - Rendering: `drawPlane`, `drawBossShip`, `draw()`.
  - `loop(now)`: the single `requestAnimationFrame` loop. It clamps `dt` to 0.05 s, then calls `update` and `draw` only while `running && !paused`.

Keep the game in this one file unless the user asks for a restructure.

## Conventions

- Match the existing style: 2-space indentation, single quotes, `const` and `let`, arrow functions for callbacks, and short section comments in the form `// ---------- Name ----------`.
- Tuning values are inline object literals near the top of the script. Change balance there rather than in the logic.
- Canvas logical size is `BASE_W` × `BASE_H` (480 × 720). `resize()` scales it to fit the viewport. Gameplay coordinates should use the base size, not the CSS size.
- Persistent settings go through `localStorage` under the `skyfighter.*` keys (`bestScore`, `muted`, `difficulty`, `reducedMotion`). Wrap new reads and writes so the game still works when storage is unavailable.
- Accessibility matters here: respect `prefers-reduced-motion` (it sets the initial state of the motion toggle), keep `announce()` for screen-reader messages, and keep the touch controls usable on phones.
- Do not add external scripts, fonts, or network calls. The game is meant to work offline from a single file.

## Verifying changes

There are no automated tests. After editing:

1. Check that the inline script parses, for example by extracting it with a short script and running `node --check`.
2. Open the page in a browser, or use the `run` skill, and play through at least one stage and one boss.
3. Check the HUD, the overlay, and the difficulty buttons at both desktop and phone widths.

## Git

- Develop on the branch assigned for the session. Do not push to other branches without permission.
- Keep commits focused and write clear messages that say what changed and why.
