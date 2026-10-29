# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a small static web project. Its only app is **Sky Fighter**, a
Raiden-style vertical shoot-'em-up that runs in the browser as a single HTML5
canvas page.

| File        | Purpose                                                        |
|-------------|----------------------------------------------------------------|
| `index.html`| The whole game: markup, CSS (`<style>`), and JS (`<script>`).  |
| `README.md` | One-line placeholder. Keep it accurate if you expand it.       |

There is no build step, no package manager, no bundler, no test suite, and no
linter configured. Do not add tooling unless the task asks for it.

## Running the game

Open `index.html` directly in a browser, or serve the directory with any static
server (for example `python3 -m http.server` from the repo root, then visit
`http://localhost:8000`). Nothing needs to be installed.

Because there are no automated tests, verify changes by loading the page and
playing through the affected behavior (menu, a stage, a boss, game over). Check
the browser console for errors.

## Architecture of `index.html`

- **Single file.** Keep everything in `index.html` unless the user asks for a
  split. Do not introduce external CDN scripts or fonts without a reason.
- **One IIFE.** All game code lives inside `(function () { 'use strict'; ... })();`
  so nothing leaks onto `window`. Keep new code inside it.
- **Canvas rendering.** A fixed logical size (`BASE_W = 480`, `BASE_H = 720`) is
  scaled to the viewport by `resize()`. Coordinates in game logic use the base
  size, not CSS pixels.
- **Main loop.** `loop(now)` clamps `dt` to 0.05 s, then calls `update(dt)` and
  `draw()` only while `running && !paused`. Game-state changes belong in
  `update`; drawing belongs in `draw` and its helpers (`drawPlane`,
  `drawBossShip`, `drawClouds`).
- **Tunable data is grouped at the top of the script.** Prefer editing these
  tables over hard-coding values in logic:
  - `DIFFICULTIES`: spawn rates, enemy speed and fire chance, lives, bombs, boss HP multiplier.
  - `KILLS_PER_STAGE`: kills needed to advance a stage.
  - `THEMES`: sky, cloud, and sun palettes per stage (cycled).
  - `ENEMY_TYPES`: `grunt`, `interceptor` (weaves), `tank` (4 HP). `minStage` gates when each appears.
  - `BOSSES`: six bosses (`VANGUARD` through `TITAN`), each with colors and `hpMul`.
  - `BOSS_PATTERNS`: bullet-pattern functions (radial, aimed spread, columns, spiral, cross, homing, double ring).
- **Persistence.** Settings and the best score use `localStorage` under keys
  prefixed `skyfighter.` (`bestScore`, `muted`, `difficulty`, `reducedMotion`).
  Every access must tolerate storage being unavailable or blocked.
- **Audio.** Procedural Web Audio (`beep`, `noiseBurst`). `ensureAudio()` must
  run from a user gesture before audio plays. Respect the `muted` setting.
- **Accessibility.** There is a live region (`srLive`) for announcements via
  `announce()`, and a reduced-motion setting that gates screen shake and
  similar effects. New visual effects should honor it.

## Controls (for reference when testing)

- Keyboard: arrow keys or WASD to move, `Space` to fire, `X` / `Shift` for a
  bomb, `P` / `Escape` to pause. `Space` is `preventDefault`-ed so the page does
  not scroll.
- Mouse: move the pointer over the canvas to steer; hold the button to fire.
- Touch: on-screen joystick (`moveZone`), fire button, and bomb button.

## Conventions

- Match the existing style in `index.html`: two-space indentation, single
  quotes in JS, `const` by default, `let` only when reassigned, concise
  one-line objects for table entries.
- Keep comments sparse and match the density already in the file.
- Use the existing helper functions (`addScore`, `spawnExplosion`,
  `screenShake`, `showStageBanner`, and so on) rather than duplicating their logic.
- When adding a new enemy, boss, or theme, extend the existing data tables and
  the code paths that read them, instead of adding special cases.
- Preserve the mobile layout and touch controls when changing HUD or overlay
  markup. The page uses `viewport-fit=cover` and a `user-scalable=no` viewport.

## Git workflow

- Develop on the branch assigned for the task (currently
  `claude/gifted-tesla-wyynif`). Do not push to `main` unless explicitly asked.
- Use `git push -u origin <branch>` when pushing.
- Do not open a pull request unless the user explicitly asks for one.
- Write commit messages in the imperative mood with a short summary line,
  followed by a body explaining the change when it is not obvious from the diff.
