# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo contains **Sky Fighter**, a single-file, browser-based vertical shoot-em-up (Raiden-style) game. There is no build step, no bundler, no package manager, and no test suite.

- `index.html` (~1,670 lines): the entire game. Inline `<style>` (lines ~10–361), HTML markup for the HUD, overlay and touch controls (lines ~362–421), and one inline `<script>` IIFE (lines ~422–1668) containing all game logic.
- `README.md`: a one-line placeholder. No setup instructions exist.

## Running and testing

- Open `index.html` directly in a browser, or serve the directory with any static server (e.g. `python3 -m http.server`).
- There are no automated tests, linters, or build commands configured. Verify changes by playing the game in a browser (desktop and mobile-sized viewports).

## Code architecture (inside `index.html`)

The script is wrapped in `(function () { 'use strict'; ... })()`, so nothing is exposed on `window`. Keep new code inside that wrapper.

Major sections, in file order:

1. **DOM references and constants**: cached `getElementById` lookups, `STORAGE_*` localStorage keys (prefixed `skyfighter.`).
2. **Adaptive performance** (`trackPerf`): averages frame times over 45 frames and steps a `quality` level (0–2) up or down to shed effects on slow devices.
3. **Tuning data**:
   - `DIFFICULTIES` (easy / normal / hard): spawn rates, enemy speed, shoot chance, starting lives and bombs, boss HP multiplier.
   - `KILLS_PER_STAGE` (18) and `THEMES` (6 sky palettes, cycled per stage).
   - `ENEMY_TYPES` (grunt, interceptor, tank) and `BOSSES` (6 bosses with colors and HP multipliers).
4. **Audio**: `ensureAudio` creates the AudioContext lazily on first user gesture. `beep` and `noiseBurst` synthesize all sound effects. There are no audio files.
5. **HUD helpers**: `renderHearts`, `renderBombs`, `showComboPop`, `showStageBanner`, `screenShake`, `flashHit`, `flashBomb`, and `announce` (screen-reader live region).
6. **Input**: keyboard (`keydown`/`keyup` into a `keys` map), mouse/pointer steering and firing via `canvasPointFromEvent`, and touch joystick plus FIRE/BOMB buttons.
7. **Game state and loop**:
   - `resetGame`, `resize` (fixed 480×720 logical canvas, scaled to fit the window up to 1.6×).
   - `update(dt)` handles movement, firing, spawning, collisions, and scoring. `draw()` renders everything.
   - `loop(now)` is the `requestAnimationFrame` driver. `dt` is capped at 0.05 s.
8. **Weapons and power-ups**: `fireWeapon` (vulcan and laser, up to level 3), `applyPowerUp` (life, bomb, weapon upgrade), `useBomb`.
9. **Bosses**: `BOSS_PATTERNS` is an array of three attack functions per boss (`fireRadial`, `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`, `fireCross`, `fireHomingSeeded`, `fireDoubleRing`). `updateBoss` cycles through them; `killBoss` and `endGame` finish the run.

## Controls (keep the in-game hint in sync)

- Keyboard: Arrows/WASD move, Space fires, X or Shift bombs, P or Esc pauses.
- Mouse: move to steer, click to fire.
- Touch: drag on the left to fly, FIRE and BOMB buttons on the right.

The on-screen hint is `#controlsHint` in `index.html`. Update it whenever a control changes.

## Persistence

Only `localStorage` is used, under these keys:

- `skyfighter.bestScore`
- `skyfighter.muted`
- `skyfighter.difficulty`
- `skyfighter.reducedMotion`

Reads must tolerate missing or invalid values. The reduced-motion default follows `prefers-reduced-motion` until the player sets it.

## Conventions

- Keep everything in `index.html` unless the user asks for a split. Do not add a build system, framework, or dependencies without asking.
- Match the existing style: 2-space indentation, `const`/`let`, arrow callbacks, short section comments using `// ---------- Name ----------`.
- Game balance lives in the constant tables above. Prefer tuning those over hard-coding numbers in logic.
- Respect accessibility hooks: announce important state changes through `announce()`, and honor the reduced-motion toggle when adding screen shake, flashes, or pulses.
- Keep the `quality` level in mind when adding particles or expensive effects. Gate them on quality where practical.
- Support both mouse/keyboard and touch when adding new input.
- Keep the layout working at phone widths. The canvas scales with `resize()`.

## Git workflow

- Develop on the branch assigned for the session (currently `claude/gifted-tesla-w1r3vi`). Do not push to other branches without explicit permission.
- Write clear, descriptive commit messages.
- Do not open a pull request unless explicitly asked.
