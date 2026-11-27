# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a small static web project. Its only substantial content is **Sky Fighter**, a Raiden-style vertical shoot-'em-up built as a single-file HTML5 canvas game.

| File | Purpose |
| --- | --- |
| `index.html` | The entire game: markup, CSS, and JavaScript in one ~1,670-line file. |
| `README.md` | Minimal project readme (title only). |

There is no package manager, build step, bundler, test suite, linter, or CI configuration. Do not add one unless asked.

## Running the game

No build is required. Open the file directly in a browser:

```sh
xdg-open index.html        # Linux
open index.html            # macOS
```

For a local server (useful if you later add features that need one):

```sh
python3 -m http.server 8000
```

Then visit `http://localhost:8000/`.

## Architecture of `index.html`

The file is organized top to bottom as:

1. **`<head>`**: meta tags, an inline SVG favicon, and a `<style>` block with a dark color scheme. The viewport is locked (`user-scalable=no`, `overflow: hidden`, `touch-action: none`), so the game fills the screen.
2. **`<body>`**: the HUD (score, best, multiplier, weapon, lives as hearts, bombs), overlays (title, pause, game over), a boss health bar, a vignette and flash layer for hit feedback, touch controls (joystick, FIRE, BOMB), and a `<canvas id="gameCanvas">`.
3. **One IIFE `<script>`** wrapping everything in `'use strict'`. Nothing is exposed on `window`, so keep new code inside that closure.

### Key constants and systems

- **Persistence**: `localStorage` keys are namespaced `skyfighter.*` (`bestScore`, `muted`, `difficulty`, `reducedMotion`). Every access should tolerate storage being unavailable or throwing.
- **Difficulty**: `DIFFICULTIES` sets spawn rates, enemy speed, shoot chance, starting lives and bombs, and boss HP multiplier for `easy`, `normal` and `hard`.
- **Stages**: `KILLS_PER_STAGE` (18) is the kill count that triggers the stage's boss warning. `THEMES` cycles background palettes per stage.
- **Enemies**: `ENEMY_TYPES` defines `grunt`, `interceptor` (weaving), and `tank` (4 HP), with `minStage` gating when each appears.
- **Bosses**: `BOSSES` gives six bosses with colors and HP multipliers. Each uses patterns from `BOSS_PATTERNS` (radial, aimed spread, columns, spiral, cross, homing, double ring).
- **Power-ups**: `life`, `bomb`, and `weapon` are handled in `applyPowerUp`. Weapons are Vulcan and Laser, each with three power levels.
- **Score**: `addScore` applies a kill-chain multiplier, which `breakChain` resets.
- **Performance**: `trackPerf` samples frame times and steps a `quality` level (0 full, 1 medium, 2 low) to hold frame rate on slow devices.
- **Accessibility**: `announce()` writes to an ARIA live region. `reducedMotion` (from the OS preference or the saved setting) disables screen shake and similar effects.
- **Main loop**: `loop(now)` calls `update(dt)` then `draw()` each frame. `resize()` scales the fixed `BASE_W × BASE_H` (480 × 720) playfield to the viewport.

### Controls

| Action | Keyboard | Touch / mouse |
| --- | --- | --- |
| Move | Arrow keys or WASD | Virtual joystick |
| Fire (hold) | Space | FIRE button; mouse-down on canvas (also steers toward the pointer) |
| Smart bomb | X or Shift | BOMB button |
| Pause | P or Escape | Pause button |
| Mute / reduce motion | Buttons in the HUD | Same |

## Conventions

- **Single file for the game.** Keep HTML, CSS and JS in `index.html` unless the user asks to split it. If you do split it, update this document.
- **No external dependencies.** The game uses only browser APIs (Canvas 2D, Web Audio, `localStorage`). Do not add CDN scripts or npm packages without asking.
- **Plain ES2017+ JavaScript.** Keep the existing style: `const`/`let`, arrow functions, and two-space indentation.
- **Keep the IIFE closed.** New helpers and state belong inside the IIFE.
- **Tune through constants.** Prefer changing the config objects (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `THEMES`) over hard-coding numbers in logic.
- **Mobile first.** Test layout and input at phone widths. The game must remain playable with touch alone, and the HUD must wrap cleanly (it uses `clamp()` and `flex-wrap`).
- **Respect reduced motion.** Any new screen shake, flash, or animation should check `reducedMotion`.

## Verification

Since there is no test harness, verify changes manually:

1. Load `index.html` in a desktop browser and play a full run on each difficulty.
2. Check that score, best score, lives, bombs and weapon level update correctly, and that best score persists after reload.
3. Reach at least stage 2 to see tanks and the first boss, and confirm the boss health bar and pattern changes.
4. Test pause (P / Escape), mute, and reduced-motion toggles.
5. Resize the window or use device emulation to confirm the canvas scales and touch controls appear.
6. Check the browser console for errors.

If you add tooling (tests, linting, a build), document the commands in this file.

## Git workflow

- Develop on the branch assigned for the session (currently `claude/gifted-tesla-jmr9o3`). Do not push to other branches without permission.
- Write clear, imperative commit messages that describe the gameplay or code change.
- Do not open pull requests unless explicitly asked.
- Do not commit secrets or local settings files.
