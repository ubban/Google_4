# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a small, dependency-free front-end project. Its only substantial
artifact is **Sky Fighter**, a Raiden-style vertical shoot-em-up built as a
single self-contained HTML5 canvas page.

| File        | Purpose                                                        |
|-------------|----------------------------------------------------------------|
| `index.html`| The entire game: inline CSS (`<style>`, ~lines 10–360), markup (`<body>`, ~361–421), and one IIFE `<script>` (~422–1668) |
| `README.md` | Placeholder; contains only the project name and a timestamp     |

There is no build step, no package manager, no test suite, and no linter
configuration. Do not add tooling unless the user asks for it.

## Running the game

Open `index.html` directly in a browser, or serve the directory statically
(e.g. `python3 -m http.server`). No install step is required.

## Architecture of `index.html`

The script is a single `(function () { 'use strict'; ... })();` closure. Nothing
is exported to `window`, so keep new code inside that closure.

Major sections, in the order they appear in the script:

1. **DOM references and constants**: cached element lookups (`canvas`, HUD
   elements, overlay, touch controls) and `STORAGE_*` localStorage keys
   (`skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`,
   `skyfighter.reducedMotion`).
2. **Config tables**: `DIFFICULTIES` (spawn rates, enemy speed/shoot chance,
   lives, bombs, boss HP multiplier), `KILLS_PER_STAGE`, `THEMES` (sky
   gradients, cloud/sun colours, per-stage names), `ENEMY_TYPES`, `BOSSES`.
3. **Audio**: `ensureAudio()` lazily creates the `AudioContext`; `beep()` and
   `noiseBurst()` synthesise all sound effects. There are no audio files.
4. **HUD and feedback**: `renderHearts`, `renderBombs`, `flashHit`,
   `showComboPop`, `showStageBanner`, `screenShake`, `announce` (screen-reader
   live region).
5. **Layout and input**: `resize()` scales a fixed `BASE_W × BASE_H`
   (480 × 720) logical canvas to the viewport. Keyboard state is kept in `keys`
   keyed by `e.code`; touch uses a joystick (`joystick*`) plus fire/bomb buttons.
6. **Game state**: `resetGame()` initialises all mutable state. Entities are
   plain objects in arrays (enemies, bullets, power-ups, explosions, boss).
7. **Simulation**: `update(dt)` advances everything; `spawnEnemy`, `spawnBoss`,
   `fireWeapon`, `useBomb`, `applyPowerUp`, `updateBoss`, `loseLife`,
   `killBoss`, `endGame`, `togglePause`.
8. **Boss patterns**: `BOSS_PATTERNS` is an array of bullet-pattern functions
   (radial, aimed spread, columns, spiral, cross, homing, double ring).
9. **Rendering**: `draw()` renders each frame; `drawPlane`, `drawBossShip`,
   `drawClouds` produce the visuals procedurally with canvas paths.
10. **Main loop**: `loop(now)` uses `requestAnimationFrame`, clamps `dt` to
    0.05 s, and calls `update` and `draw` only while running and not paused.

### Controls

- Keyboard: arrows/WASD for movement (via `keys`, keyed by `e.code`), Space to
  fire, `X` / Shift to bomb, `P` / Escape to pause.
- Mouse: move to steer, click to fire (`canvasPointFromEvent`).
- Touch: joystick in the left zone, fire and bomb buttons. Touch controls are
  shown only when `isTouchDevice` is true.

### Persistence

Best score, mute state, difficulty, and reduced-motion preference are stored in
`localStorage`. Every access must tolerate storage being unavailable (private
mode, blocked site data). Keep the existing key names stable so saved values
survive upgrades.

## Conventions

- **Single file.** Keep the game in `index.html` unless the user asks for a
  split. Do not introduce a bundler, framework, or npm dependencies.
- **Plain JavaScript, ES2015+.** Use `const`/`let`, arrow functions, and the
  existing `'use strict'` closure. Match the surrounding style: 2-space
  indentation, single quotes, semicolons.
- **Data-driven tuning.** Balance changes belong in `DIFFICULTIES`, `BOSSES`,
  `ENEMY_TYPES`, or `BOSS_PATTERNS` rather than as magic numbers scattered in
  logic.
- **Frame-rate independence.** Movement and timers use `dt` (seconds). Do not
  assume 60 fps.
- **Accessibility.** Respect the reduced-motion setting when adding screen
  shake, flashes, or other motion effects, and use `announce()` for important
  state changes that screen readers should hear.
- **Canvas coordinates.** Game logic works in the `BASE_W × BASE_H` logical
  space; `resize()` handles the CSS scaling. Convert pointer input with
  `canvasPointFromEvent`.

## Verifying changes

There are no automated tests. For gameplay or rendering changes:

1. Open `index.html` in a browser (or use the `run` skill) and confirm the
   affected behaviour on desktop and with touch emulation.
2. Check the browser console for errors on load and during a full run.
3. Verify that the game still starts, pauses, loses a life, uses a bomb, and
   reaches game over, and that best score persists across reloads.

## Git workflow

- Develop on the branch assigned for the task (currently
  `claude/gifted-tesla-48k21u`). Do not push to other branches without
  explicit permission.
- Write commit messages in the imperative mood with a short summary line, in
  the style of the existing history (e.g. `Add Sky Fighter: ...`).
- Do not open a pull request unless the user explicitly asks for one.
