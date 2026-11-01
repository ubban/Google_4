# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Google_4 is a small, dependency-free web project. Its only game is **Sky Fighter**, a Raiden-style vertical shoot-'em-up, implemented as a single self-contained HTML5 canvas page:

- `index.html` (~1,670 lines): all markup, CSS (`<style>` at ~line 10), and JavaScript (`<script>` at ~line 422) in one file.
- `README.md`: a one-line title only.

There is no build step, no package manager, no bundler, no test suite, and no linter configured. Do not add `package.json`, a framework, or a build pipeline unless the user asks for one.

## Running and verifying changes

- Open `index.html` directly in a browser. No server is required.
- To check a change, load the page and play through it: start a run, fire, use a bomb, pause/resume, and reach a boss if the change touches combat, stages, or power-ups.
- Test both desktop (keyboard/mouse) and touch layouts when changing input, HUD, or the overlay, since the page has separate mobile controls (`#touchControls`).
- Check for JavaScript errors in the browser console after any change.

## Code structure (inside `index.html`)

The game logic lives in one IIFE. Key areas, roughly in file order:

- **DOM references**: `const` lookups by element ID at the top of the script (`canvas`, `overlay`, `hud` elements, touch controls, boss bar).
- **Storage keys**: `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion`. Changing a key abandons saved player data, so treat renames as a breaking change.
- **Tuning tables**:
  - `DIFFICULTIES` (`easy` / `normal` / `hard`): spawn rates, enemy speed, enemy fire chance, starting lives and bombs, boss HP multiplier.
  - `KILLS_PER_STAGE`: kills needed to advance a stage.
  - `THEMES`: per-stage sky palettes and names.
  - `ENEMY_TYPES` (`grunt`, `interceptor`, `tank`): size, HP, speed, score, and the `minStage` each appears at.
  - `BOSSES`: name, colors, and `hpMul` for each boss.
- **Audio**: a lazily created `AudioContext` (`ensureAudio`) with synthesized sound helpers (`beep`, `noiseBurst`) and the `sfx` map. Respects the mute setting.
- **Input**: keyboard handlers on `keydown`/`keyup` keyed by `e.code`, mouse and touch handlers, and a virtual joystick for mobile.
- **Game state and update**: `resetGame()`, `update(dt)`, `updateBoss(dt)`, weapon firing (`fireWeapon`), bomb use (`useBomb`), power-ups (`applyPowerUp`), and boss attack patterns (`fire*` helpers).
- **Rendering**: `draw()`, plus drawing helpers `drawPlane`, `drawBossShip`, and `drawClouds`.
- **Main loop**: `loop(now)` drives `requestAnimationFrame`, with `dt` capped and frame-time tracking via `trackPerf`.

Keep new behavior inside this structure: add constants to the tuning tables rather than hard-coding numbers in update logic, and reuse the existing helpers (`announce` for screen-reader text, `screenShake`, `flashHit`, `spawnExplosion`) instead of duplicating them.

## Controls (keep the on-screen hint accurate)

The hint text is in `#controlsHint` (~line 417). Update it whenever bindings change.

- Keyboard: Arrows or WASD to move, Space to fire, X or Shift for a bomb, P or Escape to pause.
- Mouse: move to steer, click to fire.
- Touch: drag on the left to fly, tap FIRE and BOMB.

## Conventions

- Keep everything in `index.html` unless the user asks for a split. Inline CSS and JS are intentional.
- Use vanilla JavaScript and the existing style: two-space indentation, `const`/`let`, and small named functions.
- Preserve accessibility hooks: the `srLive` live region, `prefers-reduced-motion` handling, and keyboard reachability of the overlay buttons.
- Keep the page working at phone width (`viewport-fit=cover`, safe-area insets, `touch-action: none` on the canvas). Do not add horizontal scrolling.
- Do not add external network dependencies, CDN scripts, or analytics without an explicit request.

## Git workflow

- Develop on the branch the session names. Do not push to other branches without permission.
- Commit with clear messages that describe the gameplay or code change.
- Do not open a pull request unless the user explicitly asks for one.
