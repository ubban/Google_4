# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Google_4 is a small static web project. Its only real content is **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs as a single-page HTML5 canvas app.

- `index.html` (~58 KB): the entire game. CSS is in a `<style>` block, markup is in the body (including a `<canvas id="gameCanvas">` and the HUD/overlay elements), and all logic is in one `<script>` block wrapped in an IIFE.
- `README.md`: a placeholder title only.

There is no build step, package manager, bundler, framework, or test suite. Do not add one unless asked.

## Running and verifying

- Open `index.html` directly in a browser, or serve the directory with any static server (for example `python3 -m http.server`).
- There are no automated tests or linters configured. Verify changes by loading the game in a browser and playing through the affected path: title screen, a normal run, the boss fight, pause, game over, and mobile touch controls when touching input code.
- A headless Chromium is available in the cloud environment for this kind of check. Use the Playwright setup described in the environment instructions if you need automation.

## Code layout inside `index.html`

The script is organized in these sections, roughly in this order:

1. **DOM refs**: `document.getElementById` lookups for every HUD element and control.
2. **Storage keys**: `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion`. Persisted values are read at startup and written when they change. Keep these keys stable, since they hold players' saved state.
3. **Tuning tables** (edit these to rebalance; they are the primary knobs):
   - `DIFFICULTIES`: spawn rates, enemy speed and shooting chance, starting lives and bombs, and boss HP multiplier for easy, normal, and hard.
   - `KILLS_PER_STAGE`: kills needed to advance a stage.
   - `THEMES`: sky gradients and palette per stage, cycled as stages advance.
   - `ENEMY_TYPES`: `grunt`, `interceptor`, `tank`, with size, HP, speed, score, and the minimum stage they appear in.
   - `BOSSES`: name, colors, and `hpMul`. Boss attack patterns are in `BOSS_PATTERNS` (radial, aimed spread, columns, spiral, cross, homing, double ring).
4. **Audio**: a lazily created `AudioContext` (`ensureAudio`), `beep` and `noiseBurst` helpers, and an `sfx` object. Audio starts only after a user gesture.
5. **Input**: keyboard (`keys` map on `keydown`/`keyup`), mouse on the canvas, and touch joystick plus fire and bomb buttons.
6. **Game state and loop**: `resetGame`, `update(dt)`, `draw()`, and `loop(now)` driven by `requestAnimationFrame`. Entity updates and drawing are kept separate, so keep that split when adding things.
7. **Adaptive quality**: `trackPerf` lowers the `quality` level (0 full, 1 medium, 2 low) when frame time is high, and raises it back when there is headroom. Respect `quality` when adding expensive effects.
8. **Scaling**: a fixed `BASE_W = 480`, `BASE_H = 720` logical resolution, fitted to the window by `resize()`.

## Conventions

- Keep everything in `index.html` unless there is a clear reason to split it. Do not introduce dependencies or external CDN scripts.
- Keep the existing style: `const` and `let`, arrow functions for callbacks, two-space indentation, and section comments such as `// ---------- Name ----------`.
- Accessibility features are part of the game and should keep working: `announce()` feeds a screen-reader live region, `reducedMotion` disables screen shake and similar motion, and the mute and pause controls must stay keyboard- and touch-reachable.
- When you add a new persisted setting, use a `skyfighter.*` key and guard reads so a missing or invalid value falls back to a default.
- Preserve the mobile layout. The viewport meta disables user zoom, and touch controls are shown for touch devices, so check both desktop and touch paths when changing input or HUD code.

## Git

- Work on the branch you were assigned. Do not push to other branches without explicit permission.
- Write descriptive commit messages. Recent history uses a short imperative subject followed by a body that explains what changed and why.
- Do not open a pull request unless the user asks for one.
