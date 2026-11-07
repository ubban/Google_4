# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo is a small static web project. Its only app is **Sky Fighter**, a
Raiden-style vertical shoot-'em-up that runs in the browser on an HTML5 canvas.

- `index.html` — the entire game: markup, CSS (`<style>`), and JavaScript
  (`<script>`) in one self-contained file (~1,670 lines).
- `README.md` — minimal project readme.

There is no build step, no package manager, no bundler, no test suite, and no
linter configured. Do not add one unless asked.

## Running the game

Open `index.html` directly in a browser, or serve the directory with any static
server, for example:

```bash
python3 -m http.server 8000   # then visit http://localhost:8000
```

## Architecture of `index.html`

The script is one IIFE (`(function () { 'use strict'; ... })();`). Everything
is private to it; there are no globals or modules.

Major sections, roughly in file order:

- **DOM references** — cached `getElementById` lookups at the top of the IIFE.
- **Persistence keys** — `STORAGE_*` constants (`skyfighter.bestScore`,
  `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion`)
  read/written through `localStorage`.
- **Tunables** — `DIFFICULTIES` (spawn rate, enemy speed/fire chance, lives,
  bombs, boss HP multiplier), `KILLS_PER_STAGE`, `THEMES` (per-stage sky/cloud
  palettes), `ENEMY_TYPES`, `BOSSES`.
- **Audio** — a Web Audio synth (`ensureAudio`, `beep`, `noiseBurst`). Audio
  must be started from a user gesture; `ensureAudio()` is called on input.
- **HUD helpers** — hearts, bombs, combo pops, stage banner, screen shake,
  hit/bomb flashes, `announce()` for the screen-reader live region.
- **Input** — keyboard (`keys` map keyed by `e.code`), mouse/pointer on the
  canvas, and on-screen touch controls (virtual joystick, fire and bomb
  buttons). Touch controls are shown only on touch devices.
- **Game state & logic** — `resetGame`, `update(dt)`, `spawnEnemy`,
  `spawnBoss`, power-ups (`maybeSpawnPowerUp`, `applyPowerUp`), weapons
  (`fireWeapon`, weapon levels), bombs (`useBomb`), scoring with a kill-chain
  multiplier (`addScore`, `breakChain`), lives (`loseLife`), `endGame`,
  `togglePause`.
- **Boss attacks** — `BOSS_PATTERNS` is an array of pattern sequences built
  from bullet helpers (`fireRadial`, `fireAimedSpread`, `fireColumns`,
  `fireSpiralBurst`, `fireCross`, `fireHomingSeeded`, `fireDoubleRing`).
  `updateBoss` and `killBoss` drive them.
- **Rendering** — `draw()` renders everything each frame; `drawPlane`,
  `drawBossShip`, and `drawClouds` are the sprite-level helpers. `resize()`
  scales a fixed `BASE_W × BASE_H` (480 × 720) logical canvas to the viewport.
- **Main loop** — `loop(now)` is a `requestAnimationFrame` loop. `dt` is
  clamped to 50 ms. `update` and `draw` run only while `running && !paused`.

### Controls

| Action         | Keyboard                              | Touch            |
| -------------- | ------------------------------------- | ---------------- |
| Move           | Arrow keys or WASD                    | Virtual joystick |
| Fire (hold)    | Space                                 | Fire button      |
| Bomb           | X or Shift                            | Bomb button      |
| Pause          | P or Escape                           | Pause button     |

Mute, difficulty, and reduced-motion are set from on-screen buttons.

## Conventions

- **Single file.** Keep the game in `index.html` unless the user asks for a
  split. Do not introduce frameworks or external dependencies; the game is
  plain HTML/CSS/JS.
- **Match existing style.** Two-space indentation, single quotes in JS,
  `const`/`let` (no `var`), arrow functions for callbacks, and `'use strict'`
  inside the IIFE.
- **Keep the IIFE.** New code goes inside the existing IIFE so nothing leaks
  to `window`.
- **Tune through constants.** Balance changes (speeds, spawn rates, HP, boss
  patterns) belong in the tunables blocks rather than scattered magic numbers.
- **Frame-rate independence.** Movement and timers use `dt` (seconds); do not
  advance state per frame without scaling by `dt`.
- **Accessibility.** Respect the reduced-motion setting when adding screen
  effects, and route important status text through `announce()`.
- **Mobile.** Changes to layout or input must keep the game playable on
  phone-width screens and keep the touch controls working. Touch controls are
  enabled when `ontouchstart` or `navigator.maxTouchPoints` is present.
- **Persistence.** Any new saved setting gets a `skyfighter.*` key and must
  tolerate missing or invalid `localStorage` values.

## Verifying changes

There are no automated tests. For gameplay or UI changes:

1. Open `index.html` in a browser (or serve it locally) and check the console
   for errors.
2. Play through at least one stage: movement, firing, a power-up, a bomb, and
   a boss fight if the change touches combat or bosses.
3. Check pause/resume, game over, and the best-score persistence after reload.
4. For layout changes, check a narrow (phone-width) viewport.

If you cannot run a browser in the environment, say so explicitly and verify by
reading the code.

## Git

- Develop on the branch the task specifies. The current working branch is
  `claude/gifted-tesla-vq61wt`; `main` is the base.
- Write clear, imperative commit messages that describe the gameplay or code
  change.
- Do not open a pull request unless explicitly asked.
