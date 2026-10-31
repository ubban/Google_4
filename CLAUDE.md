# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a small static web project. Its only game is **Sky Fighter**, a
Raiden-style vertical shoot-'em-up, implemented as a single self-contained file:

- `index.html` (~1,700 lines): all markup (`<style>` in the head, `<canvas id="gameCanvas">`
  and the overlay/HUD elements in the body), and one inline `<script>` wrapped in an IIFE
  with `'use strict'`.
- `README.md`: a placeholder. It contains only the project name and a timestamp line.

There is no build step, no package manager, no bundler, no test suite, and no linter
configuration. Nothing needs to be installed.

## Running and testing

- Open `index.html` directly in a browser, or serve the folder with any static server
  (for example `python3 -m http.server`).
- There are no automated tests. Verify changes by playing the game in a browser. For
  gameplay changes, check at least: start/restart, firing, bombs, power-ups, stage
  progression, boss fights, pause, mute, and difficulty switching.
- Check both desktop (keyboard) and touch layouts when changing input or HUD code. Touch
  controls live in `#touchControls` and are shown on touch devices.

## Code layout inside `index.html`

The script is organized as plain functions and `const`/`let` state inside one IIFE. Key areas,
roughly in file order:

- **Setup and DOM refs** (top of script): cached `getElementById` lookups.
- **Config tables**: `DIFFICULTIES` (spawn rates, enemy speed, lives, bombs, boss HP
  multiplier), `KILLS_PER_STAGE`, `THEMES`, `ENEMY_TYPES`, `BOSSES`, and `BOSS_PATTERNS`
  (bullet pattern generators). Balance changes usually belong here, not in the update loop.
- **Persistence**: `localStorage` keys are prefixed `skyfighter.` (`bestScore`, `muted`,
  `difficulty`, `reducedMotion`). Every read and write must stay inside the existing guards
  and must not throw if storage is unavailable.
- **Audio**: a Web Audio synth (`ensureAudio`, `beep`, `noiseBurst`). Sound is generated, not
  loaded from files. Audio starts only after a user gesture, so call `ensureAudio()` from
  input handlers.
- **Accessibility**: `announce()` writes to the `#srLive` live region. Reduced motion is
  honored from `prefers-reduced-motion` and can be toggled by the player; screen shake and
  flashes respect it.
- **Input**: keyboard state is tracked in `keys[e.code]` on `window` keydown/keyup. Touch uses a
  joystick (`joystickStart/Move/End`) plus fire and bomb buttons. Pointer coordinates are
  mapped through `canvasPointFromEvent`.
- **Layout**: `resize()` scales a fixed `BASE_W × BASE_H` (480 × 720) logical canvas to the
  viewport. Game coordinates are always in base units, so do not hard-code device pixels.
- **Game loop**: `resetGame()`, then `update(dt)` for simulation, `draw()` for rendering,
  and `loop(now)` driving `requestAnimationFrame`. `trackPerf(dt)` monitors frame time.
  Keep simulation in `update` and drawing in `draw`.
- **Combat and state**: `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`, `applyPowerUp`,
  `fireWeapon`, `useBomb`, `addScore` (combo multiplier via `breakChain`), `loseLife`,
  `killBoss`, `endGame`, `togglePause`.

## Conventions

- Keep the project as one HTML file unless the user asks for a split. Do not add a build
  system, framework, or dependencies without a clear request.
- Use vanilla JavaScript (ES2015+) and the existing style: 2-space indentation, single
  quotes, `const` by default, and small named functions.
- Add new tunable numbers to the config tables near the top of the script rather than
  scattering magic numbers through the update or draw code.
- Keep new `localStorage` keys under the `skyfighter.` prefix and guard them the same way the
  existing ones are.
- Preserve accessibility hooks when adding UI: announce important state changes through
  `announce()`, and keep controls keyboard- and touch-reachable.
- Don't remove or rename DOM ids that the script depends on without updating every reference.

## Git workflow

- Develop on the branch the session specifies. Do not push to other branches without
  explicit permission.
- Commit with clear, descriptive messages that explain the gameplay or code change.
- Don't open a pull request unless the user asks for one.
