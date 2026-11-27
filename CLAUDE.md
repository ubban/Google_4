# CLAUDE.md

Guidance for Claude Code when working in this repository.

## What this project is

**Sky Fighter** — a Raiden-style vertical shoot-'em-up that runs in the browser. The whole game lives in one file, `index.html` (~1,670 lines: inline CSS in `<style>`, inline JavaScript in `<script>`, a single `<canvas id="gameCanvas">`). `README.md` is a placeholder.

There is no build step, no package manager, no bundler, no test suite, and no linter. Do not add any of these unless asked.

## Running it

- Open `index.html` directly in a browser, or serve the directory with any static server (e.g. `python3 -m http.server`).
- For a quick smoke test, load the page and confirm the canvas renders and the game starts without console errors.

## Code layout inside `index.html`

- **`<style>`** (around lines 10–360): dark color scheme, HUD pills (`#hud`, `.stat`), hearts/bombs display, vignette, and responsive layout. Uses `clamp()` and `env(safe-area-inset-*)` for phones.
- **Markup** (around lines 360–420): canvas, HUD, touch controls (joystick zone, fire and bomb buttons), mute/pause/motion buttons, and the stage banner.
- **`<script>`** (starts around line 422), roughly in this order:
  - Config tables: `STORAGE_*` keys, `DIFFICULTIES` (~line 485), `THEMES`, `ENEMY_TYPES`, `BOSSES`, `KILLS_PER_STAGE`.
  - Audio: Web Audio helpers (`ensureAudio`, `beep`, `noiseBurst`), with mute state persisted to `localStorage`.
  - UI helpers: `announce` (screen-reader live region), `renderHearts`, `renderBombs`, flash/shake/combo effects.
  - Input: `keydown`/`keyup` handlers, touch joystick, fire/bomb buttons, mouse handling on the canvas.
  - Game state and flow: `resetGame`, `update(dt)`, `draw()`, `loop(now)`, `loseLife`, `endGame`, `togglePause`.
  - Combat: `spawnEnemy`, `spawnBoss`, `fireWeapon`, `useBomb`, boss attack patterns (`fireRadial`, `fireAimedSpread`, `fireSpiralBurst`, `fireHomingSeeded`, and others), `updateBoss`, `killBoss`, `applyPowerUp`.
  - Rendering helpers: `drawPlane`, `drawBossShip`, `drawClouds`.

Use the function names above to navigate. Line numbers drift as the file changes, so verify them before relying on one.

## Gameplay facts worth knowing before editing

- **Difficulty** (`DIFFICULTIES`) is one of `easy`, `normal`, or `hard`. It sets spawn timing, enemy speed, enemy fire chance, starting lives, starting bombs, and boss HP multiplier. The choice is persisted under `skyfighter.difficulty`.
- **Stages**: each stage requires `KILLS_PER_STAGE` (18) kills before a boss spawns. Bosses cycle through `BOSSES` by stage index. Themes cycle through `THEMES`.
- **Enemy tiers**: `ENEMY_TYPES` entries have a `minStage`; spawning filters by current stage.
- **Persistence**: `localStorage` keys are `skyfighter.difficulty`, `skyfighter.muted`, the best score, and the reduced-motion preference. Every read and write of these should tolerate storage being unavailable (private mode, blocked site data).
- **Controls**:
  - Keyboard: move with arrow keys or WASD, fire with Space (held), bomb with X or Shift, pause with P or Escape.
  - Mouse: move with the mouse over the canvas, fire with mouse-down.
  - Touch: virtual joystick on the left, fire and bomb buttons on the right.
- **Accessibility**: there is a reduced-motion option that disables screen shake and similar effects. Keep it working when adding new effects. Game messages go through `announce()` for screen readers.

## Conventions

- Keep everything in `index.html` unless there is a clear reason to split it. Splitting would change how the project is served and should be a deliberate decision.
- Match the existing style: plain ES2015+ JavaScript, no frameworks, `const`/`let`, arrow functions for callbacks, and config values pulled into the tables near the top of the script rather than hard-coded in logic.
- Keep the game mobile-friendly. Test any HUD or layout change at phone width with no horizontal page scroll, and check that touch controls still work.
- Preserve `touch-action: none` and the `preventDefault()` calls on Space and touch buttons; removing them causes page scroll and zoom during play.
- The game loop uses a `dt`-based `update(dt)`. New movement or timers should use `dt` rather than frame counts so behavior is consistent across refresh rates.

## Verification

Since there are no automated tests, verify changes by running the game:

1. Start a static server and open the page in a desktop browser and, if the change touches layout or input, in a phone-sized viewport.
2. Start a game, confirm movement and firing, trigger a bomb, let a stage boss spawn, and check game over and restart.
3. Check the browser console for errors.
4. For audio or storage changes, confirm mute and difficulty persist across a reload.

## Git

- Development branch for this work: `claude/gifted-tesla-hewllj`.
- Commit messages should describe the change in the imperative mood. Do not create pull requests unless explicitly asked.
