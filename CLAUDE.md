# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a single-page HTML5 canvas game, **Sky Fighter**: a Raiden-style vertical shoot-'em-up. The entire app lives in one file, `index.html`, with inline `<style>` and `<script>` blocks. There is no build step, no package manager, no bundler, and no test suite.

Files:

- `index.html` — the whole game (markup, CSS, and the game script inside one IIFE).
- `README.md` — project title only.

## Running locally

Open `index.html` directly in a browser, or serve the directory with any static server (for example `python3 -m http.server`). Nothing needs to be installed.

## Architecture of `index.html`

- **Markup**: a full-screen `#container` holding `<canvas id="gameCanvas">`, a HUD (`#hud`), overlays, and touch controls (a move zone, FIRE and BOMB buttons, a mute button, and a reduced-motion button).
- **Script**: one `(function () { ... })()` IIFE. Nothing is exported to `window`, so keep new code inside it.
- **Config tables** near the top of the script, all `const`:
  - `DIFFICULTIES` — `easy` / `normal` / `hard`. Controls spawn rate, enemy speed, enemy shot chance, starting lives and bombs, and boss HP multiplier.
  - `KILLS_PER_STAGE` — kills needed to advance a stage.
  - `THEMES` — sky gradient, cloud and sun colors, and display name per stage. Cycles by stage.
  - `ENEMY_TYPES` — `grunt`, `interceptor` (weaving), and `tank` (4 HP), with minimum stage for each.
  - `BOSSES` — six bosses with colors and HP multipliers. `BOSS_PATTERNS` holds their attack patterns.
- **Persistence**: `localStorage` under the `skyfighter.*` keys (`bestScore`, `muted`, `difficulty`, `reducedMotion`). Every access should stay guarded so the game still runs when storage is unavailable.
- **Rendering**: a fixed logical size (`BASE_W = 480`, `BASE_H = 720`) scaled by `resize()`. Drawing helpers include `drawPlane`, `drawBossShip`, and the particle/explosion helpers.
- **Simulation**: `update(dt)` advances the game each frame. `updateBoss(dt)` handles boss phases. `resetGame()` restores initial state.
- **Weapons**: `fireWeapon()` handles the `vulcan` and laser weapons across three power levels. `applyPowerUp()` handles `weapon`, `bomb`, and `life` pickups. `useBomb()` clears the screen and damages bosses.
- **Scoring**: `addScore()` applies a kill-chain multiplier. `breakChain()` resets it when the player is hit.
- **Audio**: a Web Audio synth (`beep`, `noiseBurst`, and the `sfx` table). `ensureAudio()` must run from a user gesture before audio starts.
- **Input**: keyboard (arrows/WASD move, Space fire, X or Shift bomb, P or Esc pause), mouse (move to steer, click to fire), and touch (left-side joystick, FIRE and BOMB buttons). `touch-action: none` is set on the canvas and body.
- **Accessibility**: `prefers-reduced-motion` seeds the reduced-motion setting, which turns off screen shake and hit flashes. The `#srLive` live region reports game events for screen readers.

## Conventions

- Keep everything in `index.html` unless there is a clear reason to split it. If the file is split, update this section.
- Match the existing style: 2-space indentation, `const` and `let` (no `var`), arrow functions for callbacks, and short comments only where the logic is not obvious.
- Put tunable numbers in the config tables near the top rather than scattering magic numbers through the update and draw code.
- Preserve the mobile layout: the viewport meta disables zoom, and the canvas and controls must work at phone width without page scroll.
- Do not add external network dependencies. The page currently works fully offline and uses only inline assets and a data-URI favicon.

## Verifying changes

There are no automated tests or linters. To verify a change:

1. Open `index.html` in a browser and check the browser console for errors.
2. Play through at least one stage on each affected difficulty, and confirm HUD values (score, lives, bombs, stage) update.
3. For input changes, test keyboard, mouse, and touch (or a touch emulator).
4. For visual changes, check both a normal and a `reduced-motion` run.

## Git workflow

- Develop on the branch assigned for the session. Do not push to other branches without permission.
- Commit with clear, descriptive messages that explain the why when it is not obvious.
- Push with `git push -u origin <branch-name>`.
- Do not open a pull request unless explicitly asked.
