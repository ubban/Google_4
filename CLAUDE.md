# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Google_4 is a small static project. Its only real content is **Sky Fighter**, a
Raiden-style vertical shoot-em-up built as a single self-contained HTML5 canvas page.

| File        | Purpose                                                        |
|-------------|----------------------------------------------------------------|
| `index.html` | The entire game: markup and inline CSS (lines 1–421), inline JS (lines 422–end, ~1,670 lines total) |
| `README.md`  | Placeholder title only                                         |
| `CLAUDE.md`  | This file                                                      |

There is no build step, no package manager, no bundler, no test suite, and no
linter configured. Do not add one unless asked.

## Running the game

Open `index.html` directly in a browser, or serve the directory with any static
server (for example `python3 -m http.server`). No install is required.

## Architecture of `index.html`

- **Structure:** a `<canvas id="gameCanvas">`, HUD elements (score, best, multiplier,
  weapon, hearts, bombs), an overlay card for title/pause/game-over, a stage banner,
  a combo popup, a boss health bar, and touch controls.
- **Script:** one inline `<script>` wrapped in an IIFE with `'use strict'`. All state
  is module-private to that closure. Nothing is exported and nothing is global.
- **Main loop:** `loop(now)` calls `update(dt)` then `draw()` each animation frame,
  only while `running && !paused`. `dt` is clamped to 50 ms.
- **Update/draw split:** `update()` advances all entities, collisions and timers;
  `draw()` only reads state and paints. Keep new logic on the correct side of that line.
- **Difficulty:** the `DIFFICULTIES` table (`easy` / `normal` / `hard`) controls spawn
  rates, enemy speed, enemy fire chance, starting lives and bombs, and boss HP multiplier.
- **Stages and bosses:** `KILLS_PER_STAGE` kills advance the stage. `BOSSES` and
  `BOSS_PATTERNS` define boss bullet patterns (radial, aimed spread, columns, spiral,
  cross, homing, double ring).
- **Adaptive quality:** `trackPerf()` samples frame times and steps a `quality` level
  down when the frame rate drops and back up when it recovers.
- **Audio:** Web Audio API, created lazily by `ensureAudio()` on the first user gesture.
  Sounds come from `beep()` and `noiseBurst()`. Respect the mute flag.

## Persistence

`localStorage` keys, all prefixed `skyfighter.`:

- `skyfighter.bestScore`: best score, written on game over
- `skyfighter.muted`: `'1'` or `'0'`
- `skyfighter.difficulty`: `easy` / `normal` / `hard`
- `skyfighter.reducedMotion`: `'1'` or `'0'`; when unset, the page falls back to the
  OS `prefers-reduced-motion` setting

Reads and writes are not wrapped in try/catch. If you add storage access, guard it,
because private browsing or blocked storage can throw.

## Controls

- **Keyboard:** Arrows or WASD to move, Space to fire, X or Shift for bomb, P or Esc to pause.
- **Mouse / pointer:** move to steer, click to fire.
- **Touch:** drag the left zone for a virtual joystick, tap FIRE, tap BOMB.

Keep these in sync with the on-screen `#controlsHint` text.

## Conventions

- **Single file:** keep the game in `index.html` unless the user asks for a split.
  Don't introduce a framework or external dependencies.
- **Style:** match the surrounding code. It uses 2-space indentation, `const` and
  `let`, arrow functions for callbacks, and short, descriptive function names.
  Section the script with the existing grouping (state, audio, input, spawning,
  update, draw, loop).
- **Comments:** the file is lightly commented. Add a comment only where the intent
  isn't obvious from the code, such as a tuning constant or a non-obvious timing rule.
- **Accessibility:** new visual effects should honor the reduced-motion setting.
  Announce important state changes through the existing `announce()` helper, which
  feeds the `#srLive` live region.
- **Mobile:** verify layout and touch behavior at phone width. `resize()` scales the
  canvas to the viewport using `BASE_W` x `BASE_H` (480 x 720).

## Making changes

- Read the relevant section of `index.html` before editing. Edits are usually small
  and localized.
- Verify changes by loading the page in a browser and playing through the affected
  path (start, fire, bomb, a stage transition, a boss, game over). Syntax-checking
  alone doesn't cover gameplay.
- Commit with clear messages that describe the gameplay or UI change.
- Develop on the branch assigned to the session. Don't push to other branches unless asked.
