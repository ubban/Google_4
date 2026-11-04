# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo contains **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs in the browser. The entire game is a single self-contained file, `index.html`: inline CSS in `<style>`, markup in `<body>`, and one IIFE-wrapped `<script>` that holds all game logic and rendering on a `<canvas id="gameCanvas">`.

`README.md` is a placeholder and has no setup instructions.

## Repository layout

```
index.html   # The whole game: CSS, HTML, and JS (~1,670 lines)
README.md    # Placeholder
CLAUDE.md    # This file
```

There is no `package.json`, bundler, test suite, linter, or CI configuration. Do not add tooling unless asked.

## Running the game

No build step is needed. Open `index.html` directly in a browser, or serve the directory with any static server, for example:

```
python3 -m http.server 8000
```

Then visit `http://localhost:8000/index.html`.

## Architecture of `index.html`

The script runs inside a single `(function () { 'use strict'; ... })()` closure, so nothing is exposed on `window`. Within it, the code is organized in these sections, marked by `// ---------- ... ----------` comments:

- **DOM references**: `const` lookups for every element id used by the HUD, overlay, touch controls, and buttons.
- **Persistence keys**: `localStorage` keys prefixed `skyfighter.` for best score, mute, difficulty, and reduced motion. All reads go through the top-level `localStorage` calls, so wrap any new storage access in `try/catch` in case storage is blocked.
- **Adaptive performance** (`trackPerf`): a 45-frame rolling average of frame time moves a `quality` level (0 full, 1 medium, 2 low) that the renderer uses to reduce effects.
- **Tuning data** (constants near the top):
  - `DIFFICULTIES`: `easy` / `normal` / `hard`, with spawn timing, enemy speed, shoot chance, starting lives and bombs, and boss HP multiplier.
  - `KILLS_PER_STAGE` (18): kills needed to advance a stage.
  - `THEMES`: six sky palettes that cycle with stage.
  - `ENEMY_TYPES`: `grunt`, `interceptor` (weaving), and `tank` (4 HP, appears from stage 2).
  - `BOSSES`: six bosses (VANGUARD, BEHEMOTH, SPECTER, DREADNOUGHT, WRAITH, TITAN), each with colors and an `hpMul`.
  - `BOSS_PATTERNS`: bullet-pattern functions (radial, aimed spread, columns, spiral, cross, homing, double ring).
- **Audio**: a Web Audio synth. `ensureAudio()` creates the context lazily on the first user gesture. Sound effects are built from `beep()` and `noiseBurst()` and collected in an `sfx` object. Respect the `muted` flag.
- **Input**: keyboard (`keys[e.code]`), mouse (canvas `mousemove`/`mousedown` for steering and firing), and touch (a virtual joystick in `moveZone`, plus `fireBtn` and `bombBtn`).
- **Canvas sizing** (`resize`): the logical play area is `BASE_W` × `BASE_H` = 480 × 720, scaled to fit the viewport.
- **Game state and loop**:
  - `resetGame()` initializes state. `update(dt)` advances the simulation. `draw()` renders the frame.
  - `loop(now)` is driven by `requestAnimationFrame`. It clamps `dt` to 0.05 s and calls `update` and `draw` only while `running && !paused`.
- **Combat and progression**: `fireWeapon()`, `applyPowerUp()` (weapon levels), `useBomb()`, `addScore()` with a combo multiplier (`multiplier = 1 + min(9, floor(chain / 5))`; the chain resets via `breakChain()` after 2.2 s without a kill), `loseLife()`, `killBoss()`, `endGame()`, and `togglePause()`.
- **Accessibility**: `announce()` writes to a live region (`#srLive`). Reduced motion (`body.reduced-motion`) disables screen shake, flashes, and pulses. It defaults to the `prefers-reduced-motion` media query and can be toggled.

## Conventions

- **Keep it one file.** New game code goes in the existing `<script>` block, and new styles go in the existing `<style>` block, unless the user asks for a split.
- **Stay inside the IIFE.** Do not add globals. Expose nothing on `window` unless there is a reason.
- **Use the tuning tables.** Add new enemy types, bosses, themes, or difficulty values to the existing constant objects rather than hard-coding numbers in logic.
- **Keep the 480×720 logical coordinate space.** Positions, sizes, and speeds are in that space and scaled by `resize`.
- **Frame-rate independence.** Multiply movement and timers by `dt` (seconds). Do not step by frame.
- **Preserve accessibility hooks.** Keep the `aria-live` announcements, the `title` attributes on icon buttons, and the reduced-motion checks when changing HUD or effects.
- **Keep touch and keyboard parity.** Any new action needs a keyboard binding, a touch control, and a mouse path where it applies. Update the `#controlsHint` text to match.
- **Style.** Match the existing code: 2-space indent, single quotes in JS, `const` by default, arrow functions for callbacks.
- **Themes cycle by stage.** Stage `n` uses `THEMES[(n - 1) % THEMES.length]`.

## Testing and verification

There is no automated test suite. To verify a change:

1. Open `index.html` in a desktop browser and play through a few stages on each difficulty.
2. Check the mobile layout with touch emulation: joystick, FIRE, BOMB, and the pause button.
3. Test pause (`P` / `Esc`), mute, reduced motion, and difficulty selection before the game starts.
4. Confirm no errors appear in the browser console.
5. Confirm the best score persists across reloads.

Run a headless browser check when it helps, for example with Playwright, and report what was actually observed.

## Git workflow

- Develop on the branch the session specifies. Commit with clear messages and push with `git push -u origin <branch>`.
- Do not open a pull request unless the user asks for one.
