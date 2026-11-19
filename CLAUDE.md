# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a small, dependency-free static web project. Its only real content is **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs in the browser on an HTML5 `<canvas>`.

Files:

- `index.html` — the entire game: markup, CSS (`<style>`), and JavaScript (`<script>`) in one self-contained page (~1,700 lines).
- `README.md` — placeholder title only.
- `CLAUDE.md` — this file.

There is no build step, package manager, test suite, linter, or CI configuration in the repo.

## Running the game

No tooling is required. Either:

- open `index.html` directly in a browser, or
- serve the directory with any static server, e.g. `python3 -m http.server 8000`, then visit `http://localhost:8000/`.

## Architecture of `index.html`

The game script is a single IIFE (`(function () { 'use strict'; ... })();`) with no globals. Key parts, in file order:

1. **DOM references** — cached `getElementById` lookups for HUD, overlay, touch controls, and the canvas.
2. **Persistence keys** — `localStorage` keys prefixed `skyfighter.` (`bestScore`, `muted`, `difficulty`, `reducedMotion`). All reads/writes go through these constants.
3. **Config tables** — `DIFFICULTIES` (spawn rate, enemy speed, shoot chance, lives, bombs, boss HP multiplier), `KILLS_PER_STAGE`, `THEMES` (per-stage sky/cloud/sun palettes), `ENEMY_TYPES`, `BOSSES`.
4. **Audio** — a lazily created Web Audio context (`ensureAudio`) with `beep` and `noiseBurst` helpers. Audio starts only after a user gesture.
5. **Input** — keyboard handlers on `window`, mouse/pointer steering on the canvas, and touch joystick plus FIRE/BOMB buttons for mobile.
6. **Game state and rules** — `resetGame`, `update(dt)` (movement, firing, collisions, spawning, scoring, stage progression), `applyPowerUp`, `useBomb`, `loseLife`, `endGame`, `togglePause`.
7. **Weapons and bosses** — `fireWeapon`, the `fire*` pattern helpers, `BOSS_PATTERNS`, `updateBoss`, `killBoss`.
8. **Rendering** — `draw()` plus helpers (`drawPlane`, `drawBossShip`, `drawClouds`). Rendering is pure canvas 2D; there is no sprite sheet.
9. **Main loop** — `loop(now)` calls `update` then `draw` each frame via `requestAnimationFrame`. `dt` is clamped to 50 ms so that tab switches don't cause large jumps.

Other notable pieces: the HUD (score, multiplier, weapon level, hearts, bombs, boss bar), the overlay card for start/pause/game-over, responsive canvas sizing in `resize()` against a `480×720` base resolution, and a reduced-motion option (stored in `skyfighter.reducedMotion`, defaulting to the `prefers-reduced-motion` media query) that dampens screen shake.

## Controls (for reference when changing input code)

- Keyboard: Arrows/WASD move, Space fires, X or Shift bombs, P or Esc pauses.
- Mouse: move to steer, click to fire.
- Touch: drag left zone to fly, FIRE and BOMB buttons on screen.

Note that `Space` default is prevented to stop page scrolling; keep that behavior when editing keydown handling.

## Conventions

- Keep the game in the single `index.html` unless the user asks for a split. Don't introduce a build step, bundler, or npm dependencies without asking.
- Stay with vanilla JS and the canvas 2D API. No frameworks.
- Match the existing style: `const`/`let`, arrow functions for callbacks, two-space indentation, single quotes in JS, and small named helper functions.
- Tune gameplay through the config tables (`DIFFICULTIES`, `THEMES`, `ENEMY_TYPES`, `BOSSES`, `BOSS_PATTERNS`) rather than hard-coding magic numbers in logic.
- New persisted values must use the `skyfighter.` key prefix and a named constant.
- Any new `localStorage` access should tolerate failures (private mode, disabled storage) where practical, as the existing code assumes it works; be cautious about adding more unguarded calls.
- Keep UI text that screen readers should hear routed through `announce()`.
- Preserve mobile behavior: the layout uses `touch-action: none` and safe-area insets; test touch-related changes at phone widths.

## Verifying changes

There are no automated tests. For any change to `index.html`:

1. Load the page in a browser (desktop and, if touching layout or input, a phone-sized viewport).
2. Start a game, confirm movement, firing, bombs, and pause/resume work.
3. Reach a stage boss if you changed combat, scoring, or boss logic.
4. Check the browser console for errors.

For a quick syntax check of the game logic, extract the inline `<script>` block to a scratch file and run `node --check` on it.

## Git

- Development branch for this session: `claude/gifted-tesla-boyhjg`. Commit and push there; don't push to other branches without explicit permission.
- Don't create pull requests unless the user asks.
- Keep commits focused. Describe the gameplay or UI change in the subject line.
