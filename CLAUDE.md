# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up that runs in the browser.

The whole game is a single self-contained file, `index.html`. It holds the markup, the CSS (in a `<style>` block), and the game logic (in a `<script>` block). The rest of the repo is `README.md`, which contains only the project title.

There is no build step, no bundler, no package manager, no test suite, and no linter. Do not add any of these unless the user asks.

## Running the game

Open `index.html` directly in a browser. No server is required.

For a quick local server, use `python3 -m http.server` from the repo root, then visit `http://localhost:8000`.

## Code layout (`index.html`)

Rough map of the file, from top to bottom:

- **Markup** (around lines 360–420): `#container` holds the `<canvas id="gameCanvas">`, the HUD (`#hud`), boss bar, combo pop, stage banner, top bar buttons (mute / reduced motion / pause), touch controls (`#moveZone`, `#joystickBase`, `#actionZone` with BOMB and FIRE buttons), and the start/pause overlay (`#overlay`, `#overlayCard`).
- **Styles** (top of file): CSS custom properties are not used. Colors are literal hex or `rgba()` values.
- **Config constants** (around lines 460–520):
  - `STORAGE_*` keys for `localStorage` (best score, mute, difficulty, reduced motion).
  - `DIFFICULTIES` (easy / normal / hard): spawn rates, enemy speed, shoot chance, starting lives and bombs, boss HP multiplier.
  - `KILLS_PER_STAGE` (18). Stages advance after this many kills.
  - `THEMES`: six sky palettes that cycle with the stage.
  - `ENEMY_TYPES`: `grunt`, `interceptor` (weaving), `tank` (4 HP, stage 2+).
  - `BOSSES`: six bosses (VANGUARD, BEHEMOTH, SPECTER, DREADNOUGHT, WRAITH, TITAN), each with its own colors and HP multiplier.
- **Audio** (around lines 543–660): a lazily created `AudioContext` (`ensureAudio`). Sound effects are built from `beep()` and `noiseBurst()`. Audio is unlocked on the first user gesture.
- **Input** (around lines 765–865): keyboard (`keydown`/`keyup` on `e.code`), mouse (steer and fire on the canvas), and touch (virtual joystick plus FIRE and BOMB buttons).
- **Game state and loop**: module-level `let` variables (score, lives, bombs, stage, chain, multiplier, and the entity arrays `bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`, `boss`). `resetGame()` initializes them. `update(dt)` advances the simulation, `draw()` renders it, and `loop(now)` drives them with `requestAnimationFrame`.
- **Boss patterns** (around lines 1324–1410): `BOSS_PATTERNS` is an array of bullet-pattern functions (radial, aimed spread, columns, spiral, cross, homing, double ring) that `updateBoss()` cycles through.
- **Power-ups** (around lines 1304–1323): `life`, `bomb`, and `weapon` pickups, applied by `applyPowerUp()`.
- **Performance**: `trackPerf(dt)` sets a `quality` level (0 full, 1 medium, 2 low) based on frame timing, and rendering reads it.

### Controls

- Keyboard: arrows or WASD to move, Space to fire, X or Shift to bomb, P or Esc to pause.
- Mouse: move to steer, click to fire.
- Touch: drag on the left side to fly, tap FIRE, tap BOMB.

## Conventions

- **Keep it one file.** Put new HTML, CSS, and JS into `index.html` unless the user asks for a split. If the project is split later, update this file.
- **Match the existing style.** Use 2-space indentation, single quotes in JS, `const` by default, and `let` for mutable game state. Keep the existing section comments.
- **Game constants live at the top of the script block.** Tune balance in `DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, and `THEMES` rather than hardcoding numbers in the logic.
- **Time-based movement.** Update functions take `dt` (seconds). Multiply speeds by `dt` so motion stays frame-rate independent.
- **Persist settings through the `STORAGE_*` keys**, and wrap `localStorage` access so the game still runs when storage is unavailable (private mode, blocked storage).
- **Accessibility.** Respect the reduced-motion toggle for screen shake, flashes, and pulses. Announce important state changes through `announce()` (the `#srLive` live region).
- **Mobile first.** Keep the layout working at phone widths. Preserve `touch-action: none` on the canvas and controls, and keep the `resize()` logic that fits the 480×720 base canvas to the viewport.

## Testing and verification

There are no automated tests. To verify a change:

1. Open `index.html` in a browser (or serve it locally).
2. Start a game and check the affected behavior: movement, firing, bombs, power-ups, stage transitions, and boss fights.
3. Check all three difficulties if you changed balance.
4. Check the layout at a narrow (phone) width and with touch controls.
5. Check the browser console for errors.

For browser-based checks, Playwright is available in the remote environment. Use `require('playwright')` and `chromium.launch()` without extra options.

## Git workflow

- Develop on the branch assigned to the session. Do not push to other branches without explicit permission.
- Commit only when asked, or when the session's instructions say to finish by pushing.
- Do not create pull requests unless the user explicitly asks for one.
- Do not commit secrets or local settings files.
