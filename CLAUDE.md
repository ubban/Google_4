# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo is a small static web project. Its only game is **Sky Fighter**, a
Raiden-style vertical shoot-em-up that runs as a single self-contained HTML5
canvas page.

| File        | Purpose                                                        |
|-------------|----------------------------------------------------------------|
| `index.html`| The whole game: markup (~360 lines), CSS in `<style>`, and the game script in `<script>`. |
| `README.md` | Minimal placeholder; no setup instructions yet.                |

There is no build step, no package manager, no bundler, and no test suite.
Nothing needs to be installed.

## Running the game

Open `index.html` directly in a browser, or serve the folder with any static
server (for example `python3 -m http.server` from the repo root and visit
`http://localhost:8000`). Changes take effect on reload.

## Architecture of `index.html`

The file is organized top to bottom as:

1. **`<style>`**: layout and theming for the HUD, overlay card, touch controls,
   boss bar, and screen effects (shake, hit flash, low-health vignette).
2. **Markup**: a `#container` holding the `<canvas id="gameCanvas">`, HUD
   (`#hud`, `#bossBarWrap`, `#comboPop`, `#stageBanner`), top-bar icon buttons
   (`#muteBtn`, `#motionBtn`, `#pauseBtn`), touch controls (`#touchControls`),
   and the start/pause overlay (`#overlay`, `#overlayCard`).
3. **`<script>`**: one IIFE (`'use strict'`) containing all game logic. It has
   no modules and no globals beyond the DOM.

Key parts of the script, in roughly the order they appear:

- **Persistence keys**: `skyfighter.bestScore`, `skyfighter.muted`,
  `skyfighter.difficulty`, `skyfighter.reducedMotion`. These are read from and
  written to `localStorage`. Any new persisted setting should use the same
  `skyfighter.*` prefix.
- **Tuning tables**:
  - `DIFFICULTIES` (`easy` / `normal` / `hard`): spawn rates, enemy speed and
    fire chance, starting lives and bombs, and a boss HP multiplier.
  - `KILLS_PER_STAGE` (18): kills needed before a boss spawns.
  - `THEMES`: sky gradients and cloud/sun colors, cycled by stage.
  - `ENEMY_TYPES`: `grunt`, `interceptor` (weaving), and `tank` (4 HP, from stage 2).
  - `BOSSES`: six boss designs (colors and HP multiplier), with attack patterns
    in `BOSS_PATTERNS`.
- **Audio**: `ensureAudio()` lazily creates the `AudioContext` on the first user
  gesture. `beep()` and `noiseBurst()` synthesize all sound effects; there are
  no audio files.
- **Game state and loop**: `resetGame()` initializes state. `update(dt)` runs
  game logic, `draw()` renders, and `loop(now)` drives them via
  `requestAnimationFrame`. `dt` is clamped to 50 ms. `trackPerf()` adjusts a
  `quality` level from a rolling frame-time average.
- **Gameplay systems**: `fireWeapon()` (Vulcan/Laser, 3 power levels),
  `useBomb()` (screen clear that damages bosses), `spawnEnemy()`,
  `spawnBoss()`, `updateBoss()`, `killBoss()`, `applyPowerUp()`
  (weapon level, bomb, extra life), `addScore()` with a kill-chain multiplier
  reset by `breakChain()`, and `loseLife()` / `endGame()`.
- **Rendering**: `drawPlane()`, `drawBossShip()`, `drawClouds()`, and `draw()`.
  Canvas is sized to a base logical resolution of 480×720 (`BASE_W`, `BASE_H`)
  and scaled in `resize()`.
- **Input**: keyboard (`keydown` / `keyup` on `window`), mouse on the canvas,
  and touch joystick plus FIRE and BOMB buttons. Pause is `P` or `Esc`; bomb is
  `X` or `Shift`.
- **UI feedback**: HUD updates, combo pop-ups, stage banners, and an
  `aria-live` region (`#srLive`) fed by `announce()` for screen readers.

## Conventions

- Keep everything in `index.html` unless the user asks to split it. Do not
  add a build step or dependencies without a clear request.
- Stay in the existing style: a single IIFE, `const` by default, `let` for
  mutable state, arrow functions for callbacks, two-space indentation, and
  single quotes in JavaScript.
- Put new tuning values in the existing tables (`DIFFICULTIES`, `ENEMY_TYPES`,
  `BOSSES`, `THEMES`) rather than hard-coding them in logic.
- Respect the reduced-motion setting. `reducedMotion` defaults to the
  `prefers-reduced-motion` media query and can be toggled with the top-bar
  button. Screen shake is scaled down and the hit flash is skipped when it is
  on, so new screen effects should follow the same pattern.
- Keep touch and keyboard controls working together. Any new action needs both
  a keyboard binding and a touch control if it is needed during play.
- Keep the page usable on phones: the overlay and touch controls are sized for
  small screens.

## Verifying changes

There are no automated tests or linters. To verify a change:

1. Open `index.html` in a browser and check the browser console for errors.
2. Play through a start, a stage clear, a boss fight, a bomb, a pause/resume,
   and a game over. Confirm the best score persists after a reload.
3. Toggle mute, reduced motion, and difficulty, and confirm each setting
   persists after a reload.
4. For layout changes, check a narrow (phone-width) viewport and the touch
   controls.

## Git

- The active development branch for this work is `claude/gifted-tesla-ltlsj9`.
  Push there unless told otherwise.
- Commit messages should describe the gameplay or code change in the imperative
  mood, with a short summary line and an optional body explaining the why.
