# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Google_4 contains **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs in the browser. The entire game is a single self-contained file, `index.html`, with inline CSS and one inline `<script>` that drives an HTML5 `<canvas>`. There is no build step, no bundler, no package manager, and no external dependencies.

Files:

- `index.html` — the whole game (markup, styles, and game logic).
- `README.md` — minimal project readme.
- `CLAUDE.md` — this file.

## Running the game

No install or build is needed. Open `index.html` directly in a browser, or serve the directory with any static file server, for example:

```
python3 -m http.server 8000
```

then visit `http://localhost:8000/`.

There is no test suite, linter, or CI configured. Verify changes by playing the game in a browser and exercising the affected code path.

## Architecture of `index.html`

The file has three parts, in order:

1. **`<style>`** — dark-theme CSS (`color-scheme: dark`). The HUD is a row of pill-shaped `.stat` elements at the top. The overlay card (`#overlay`) holds the title, difficulty buttons, stats, and the Start button. Touch controls (`#touchControls`, `#moveZone`, `#fireBtn`, `#bombBtn`) are shown on touch devices. A `body.reduced-motion` class disables screen shake, flashes, and pulses.
2. **Markup** — the canvas (`#gameCanvas`), HUD elements, overlay, touch controls, and a hidden `#srLive` live region used by `announce()` for screen readers.
3. **One IIFE `<script>`** — all game code lives inside `(function () { ... })();` so nothing leaks onto `window`. Keep new code inside this wrapper.

### Key sections of the script (in file order)

- **Persistence** — `localStorage` keys are prefixed `skyfighter.` (`bestScore`, `muted`, `difficulty`, `reducedMotion`). Always read and write them through these constants.
- **Adaptive quality** — `trackPerf()` averages the last 45 frame times and steps `quality` (0 full, 1 medium, 2 low) to keep the frame rate up.
- **Tuning data** — `DIFFICULTIES` (easy/normal/hard: spawn rate, enemy speed, shot chance, lives, bombs, boss HP multiplier), `KILLS_PER_STAGE`, `THEMES` (per-stage sky palettes), `ENEMY_TYPES` (grunt, interceptor, tank), and `BOSSES` (six boss designs with HP multipliers). Balance changes should usually be made here, not in logic.
- **Audio** — a lazily created Web Audio context (`ensureAudio()`), with helpers `beep()`, `noiseBurst()`, and a `sfx` table. Audio is created only after a user gesture.
- **Input** — keyboard (`keys` map keyed by `e.code`), mouse (`canvasPointFromEvent`), and touch (`joystickStart/Move/End`, fire and bomb buttons).
- **Game state** — top-level `let` variables such as `score`, `lives`, `bombCount`, `stage`, `chain`, `multiplier`, and `running`/`paused`/`gameOver`. `resetGame()` restores them for a new run.
- **Rendering** — `drawPlane`, `drawBossShip`, `drawClouds`, and `draw()`. `loop(now)` is the `requestAnimationFrame` driver, computes `dt`, and calls `update(dt)` then `draw()`.
- **Update logic** — `update(dt)` handles player movement, firing, enemy spawns, collisions, and power-ups. `updateBoss(dt)` runs boss movement and the attack patterns in `BOSS_PATTERNS`.
- **Scoring** — `addScore()` applies the kill-chain multiplier. `breakChain()` resets it when the player is hit.
- **Lifecycle** — `loseLife()`, `killBoss()`, `endGame()`, and `togglePause()`.

### Controls (from the in-game hint)

- Mouse: move to steer, click to fire.
- Arrows / WASD: move. Space: fire. X or Shift: bomb. P or Esc: pause.
- Mobile: drag on the left to fly, tap FIRE, tap BOMB.

## Conventions

- **Single file.** Keep the game in `index.html` unless the user asks for a split. Do not add a build step or npm dependencies to this project without asking.
- **Match the existing style.** The code uses `const`/`let`, arrow functions for callbacks, `addEventListener` for all events, and short comments only where a rule is not obvious. Name new tuning values alongside the existing data tables.
- **Stay inside the IIFE.** New globals must not be added to `window`.
- **Respect reduced motion.** Any new screen shake, flash, or pulse should check `reducedMotion` or use the existing `screenShake()`, `flashHit()`, and `flashBomb()` helpers.
- **Keep the game playable on phones.** Preserve the `touch-action: none` behaviour, the touch controls, and the scaling in `resize()` (base canvas 480×720).
- **Accessibility.** Announce important state changes through `announce()`, and keep focusable controls labelled with `title` or visible text.

## Git workflow

- Develop on the branch assigned for the session (currently `claude/gifted-tesla-a9q1et`). Do not push to other branches without explicit permission.
- Write clear commit messages that describe the gameplay or code change.
- Push with `git push -u origin <branch>`.
- Do not open a pull request unless the user explicitly asks for one.
