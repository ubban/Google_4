# CLAUDE.md

Guidance for Claude Code when working in this repository.

## What this repo is

`Google_4` is a small static web project. Its only real content is **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs in a browser canvas.

| File | Purpose |
| --- | --- |
| `index.html` | The entire game: markup, CSS, and JavaScript in one ~1,670-line file |
| `README.md` | Minimal project title. No setup instructions yet. |

There is no build system, package manager, bundler, test suite, or linter. Do not add one unless the user asks. Keep the game as a single self-contained HTML file that opens directly in a browser.

## Running it

Open `index.html` in a browser. No server or install step is needed. For a quick local server, use `python3 -m http.server` from the repo root and visit `http://localhost:8000`.

To verify a change, load the page in a browser and play it. Syntax errors show up in the console. Check desktop (keyboard and mouse) and a narrow or touch viewport, since the layout and touch controls are both part of the game.

## Structure of `index.html`

The file has three sections, in this order:

1. **`<head>`**: meta tags (viewport, `theme-color` `#0b1d3a`, description) and a `<style>` block with all CSS. The UI includes the HUD, the overlay for start, pause, and game over screens, boss health bar, touch controls, and a `prefers-reduced-motion` mode.
2. **`<body>`**: the `<canvas id="gameCanvas">` element plus DOM elements for the HUD (score, best, multiplier, weapon, hearts, bombs), overlays, and touch buttons. The element IDs are referenced directly from the script.
3. **`<script>`**: one IIFE (`'use strict'`) containing all game logic. Nothing is exported to `window`.

Inside the script, the code is grouped with `// ---------- Section ----------` comments. Keep that convention when adding code.

### Key constants (search for these names)

- `BASE_W = 480`, `BASE_H = 720`: logical canvas size. `resize()` scales the canvas to fit the window, capped at 1.6x.
- `DIFFICULTIES`: `easy` / `normal` / `hard`. Tunes spawn rate, enemy speed, enemy fire chance, starting lives and bombs, and boss HP multiplier.
- `KILLS_PER_STAGE = 18`: kills needed to advance a stage.
- `THEMES`: six sky palettes, one per stage, cycled in order.
- `ENEMY_TYPES`: `grunt`, `interceptor` (weaves), and `tank` (4 HP). Each has a `minStage` gate.
- `BOSSES`: six bosses (VANGUARD, BEHEMOTH, SPECTER, DREADNOUGHT, WRAITH, TITAN), each with colors and an `hpMul`. Their attack patterns are in `BOSS_PATTERNS` (radial, aimed spread, columns, spiral, cross, homing, double ring).
- Persistence keys use the `skyfighter.*` prefix in `localStorage`: `bestScore`, `muted`, `difficulty`, `reducedMotion`.

### Main game functions

- `update(dt)`: the per-frame simulation. Handles player movement, firing, enemy spawns, collisions, and power-ups.
- `draw()`: the per-frame render. Called only when the game is running and not paused.
- `loop(now)`: the `requestAnimationFrame` driver. It clamps `dt` to 0.05 s and calls `trackPerf` for the adaptive quality setting.
- `fireWeapon()`: `vulcan` (spread scales with weapon level) and `laser`.
- `applyPowerUp(type)`: `life`, `bomb`, and `weapon`. A weapon pickup raises the level from 1 to 3. At level 3 the next pickup switches between vulcan and laser and resets the level to 1.
- `useBomb()`, `loseLife()`, `endGame()`, `togglePause()`, `resetGame()`.
- `sfx`: Web Audio sound effects. Audio is initialised lazily through `ensureAudio()` on the first user gesture, because browsers block audio before one.

### Input

- Keyboard: arrows or WASD move, Space fires, X or Shift uses a bomb, P or Escape pauses.
- Mouse: move to steer and click to fire, handled on the canvas.
- Touch: a virtual joystick on the left (`moveZone`), plus FIRE and BOMB buttons.

## Conventions

- **Single file.** Put new CSS in the `<style>` block and new JS in the existing `<script>` IIFE. Do not split files or add dependencies unless asked.
- **Plain JS, no framework.** Use `const`/`let`, arrow functions, and the existing helpers (`beep`, `noiseBurst`, `spawnExplosion`, `announce`) before writing new ones.
- **Frame-rate independence.** Movement and timers take `dt` (seconds). Do not hard-code per-frame step sizes.
- **Respect the existing accessibility hooks.** The reduced-motion toggle (`reducedMotion` variable, `body.reduced-motion` class, and `motionBtn`) exists to tone down effects, so new visual effects should respect it. Keep the `srLive` announcements for important state changes, and keep the keyboard controls working alongside touch.
- **Persist with the `skyfighter.` prefix** and read storage values with a fallback, as the existing code does.
- **Tune with constants.** Put balance numbers in the existing tables (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`) rather than in logic.
- Use the existing comment style (`// ---------- Section ----------`), and keep comments brief.

## Git

- Development branch for this work: `claude/gifted-tesla-1z2tc4`. Push there with `git push -u origin <branch>`.
- Commit only when asked. Keep commits focused on a single change.
- Do not open a pull request unless the user explicitly asks for one.

## Known gaps

- `README.md` has no description, setup, or controls. Expand it if the user asks.
- No automated tests or linting exist.
