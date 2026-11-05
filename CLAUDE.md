# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Google_4 is a single-file browser game, **Sky Fighter**: a Raiden-style vertical shoot-'em-up. There is no build system, package manager, framework, or test suite.

- `index.html` (~1,670 lines) holds all markup, CSS, and JavaScript. It is an inline `<style>` block and one inline `<script>` IIFE that draws everything on a `<canvas id="gameCanvas">`.
- `README.md` contains only the repository title.

Run the game by opening `index.html` in a browser, or serve the directory with any static server.

## Repository layout

```
index.html   # The entire game (HTML + CSS + JS)
README.md    # Minimal project readme
```

Keep the game in a single file unless the user asks for a split. Do not add a build step, bundler, or dependencies without a request.

## Code structure (inside `index.html`)

The script is wrapped in one IIFE. Roughly in file order:

- **DOM refs and constants**: element lookups, `STORAGE_*` localStorage keys, `DIFFICULTIES` (`easy` / `normal` / `hard`), `KILLS_PER_STAGE`, `THEMES` (per-stage sky palettes), `ENEMY_TYPES` (grunt, interceptor, tank), `BOSSES` (per-stage boss designs, cycled with `(stage - 1) % BOSSES.length`).
- **Persistence and settings**: best score, mute, difficulty, and reduced-motion are stored in localStorage. Wrap any new storage access in try/catch if it could run in a restricted context.
- **Audio**: a lazily created `AudioContext` (`ensureAudio`) with synthesized sound via `beep` / `noiseBurst`. Named effects live in the `sfx` object.
- **Input**: keyboard (`keys` map keyed by `e.code`), touch joystick and fire/bomb buttons, and mouse (`mouseTarget` on the canvas). Touch controls are shown only on touch devices.
- **Layout**: `BASE_W = 480`, `BASE_H = 720`. `resize()` scales the canvas to fit the window, capped at 1.6x.
- **Game state**: module-level `let` variables (`player`, `bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`, `boss`, `score`, `lives`, `bombCount`, `stage`, `chain`, `multiplier`, and so on). `resetGame()` reinitializes them.
- **Update / draw**: `update(dt)` advances the simulation, `updateBoss(dt)` handles boss behavior, and `draw()` renders each frame. `loop(now)` is the `requestAnimationFrame` driver. `dt` is clamped to 0.05 s.
- **Game flow**: `endGame()`, `togglePause()`, and `useBomb()`. The pause key is `P` or `Escape`. Bomb keys are `X` or `Shift`.
- **Adaptive quality**: `trackPerf` samples frame times and lowers `quality` (0 full, 1 medium, 2 low) when the frame rate drops.

## Controls

| Action | Keyboard | Touch |
|---|---|---|
| Move | Arrow keys or WASD | Joystick (left) |
| Fire | Space (hold) | Fire button |
| Smart bomb | X or Shift | Bomb button |
| Pause | P or Escape | Pause button |
| Aim / move | Mouse over canvas | - |

## Gameplay rules to preserve

- Enemies appear from stage 1; `tank` enemies from stage 2.
- Stages advance after `KILLS_PER_STAGE` (18) kills, followed by a boss fight.
- Power-ups: `life`, `bomb`, and `weapon`. The weapon power-up toggles between `vulcan` and `laser`, and raises `level`.
- Difficulty changes lives, bomb count, spawn rate, enemy fire chance, and boss HP multiplier. Edit these in `DIFFICULTIES`, not in scattered code.

## Conventions

- Plain ES2017+ JavaScript, no modules, no transpilation. Match the existing 2-space indentation and single quotes.
- Put new tuning values in the existing constant objects (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `THEMES`) instead of hard-coding numbers in logic.
- Respect reduced-motion (`body.reduced-motion` and `reducedMotion`) when adding screen effects, and keep mobile layout working (`viewport-fit`, safe-area insets, `touch-action: none`).
- Keep the UI accessible: announce important state changes through the existing `announce()` / `srLive` live region.
- Do not add external CDN scripts or fonts without a request. The page currently uses no network resources.

## Testing and verification

There is no automated test suite or linter. To verify a change:

1. Open `index.html` in a desktop browser and play through a few kills, a stage transition, and a boss fight.
2. Check the pause overlay, the game-over screen, and the best-score persistence after a reload.
3. For layout or input changes, check a narrow (phone-width) viewport and touch emulation.
4. Watch the browser console for errors. A clean run is the minimum bar.

## Git workflow

- Develop on the branch assigned for the session (currently `claude/gifted-tesla-rnv5uk`). Do not push to other branches without permission.
- Write clear, descriptive commit messages that explain the gameplay or code change.
- Do not open a pull request unless the user asks for one.
