# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo is **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs entirely in the browser. It is a single self-contained file:

- `index.html` (~1,670 lines) — all HTML, CSS (`<style>`, ~lines 10–362), markup (`<canvas id="gameCanvas">` and HUD/overlay elements), and JavaScript (one IIFE in a `<script>` block starting ~line 422).
- `README.md` — a placeholder; it currently has no project documentation.

There is no build step, no package manager, no bundler, no test suite, and no external dependencies. The game uses only the Canvas 2D API and Web Audio.

## Running and testing

- Open `index.html` directly in a browser, or serve the directory with any static server (e.g. `python3 -m http.server`).
- There are no automated tests or linters configured. Verify changes by playing the game in a browser, covering at least: start, shooting, a bomb, pause/resume, a stage transition, a boss fight, game over, and the best-score update.
- For mobile-layout changes, check the touch controls at a narrow viewport as well as on desktop.

## Architecture (inside `index.html`)

The script is one IIFE with `'use strict'`. Key parts, in roughly the order they appear:

- **DOM references** — cached at the top (`canvas`, `overlay`, HUD elements, touch controls, buttons). Keep these in sync if you rename IDs in the markup.
- **Tuning tables** — the main place to change gameplay balance:
  - `DIFFICULTIES` (`easy` / `normal` / `hard`): spawn rate, enemy speed, enemy fire chance, starting lives and bombs, boss HP multiplier.
  - `KILLS_PER_STAGE` (18): kills needed to advance a stage.
  - `THEMES`: six sky palettes cycled per stage.
  - `ENEMY_TYPES` (`grunt`, `interceptor`, `tank`): size, HP, speed, color, score, and the `minStage` at which each appears.
  - `BOSSES`: six bosses, each with a name, colors, and HP multiplier.
  - `BOSS_PATTERNS` (~line 1372): bullet-pattern functions (radial, aimed spread, columns, spiral, cross, homing, double ring) used by `updateBoss`.
- **Persistence** — `localStorage` keys prefixed `skyfighter.` (`bestScore`, `muted`, `difficulty`, `reducedMotion`). Reads and writes should tolerate storage being unavailable.
- **Audio** — `ensureAudio()` lazily creates the `AudioContext` (browsers require a user gesture). `beep()` and `noiseBurst()` synthesize all sound effects; there are no audio files.
- **Rendering** — `draw()` renders each frame; helpers include `drawPlane`, `drawBossShip`, and `drawClouds`. `resize()` scales the fixed `BASE_W × BASE_H` (480×720) logical canvas to the window.
- **Simulation** — `update(dt)` advances the game state: player movement, bullets, enemy spawning, collisions (`rectsOverlap`), power-ups (`applyPowerUp`), bombs (`useBomb`), score and combo (`addScore`, `breakChain`). `updateBoss(dt)` drives boss phases; `killBoss()` and `loseLife()` handle transitions; `endGame()` shows the overlay.
- **Main loop** — `loop(now)` clamps `dt` to 50 ms, calls `update` and `draw` only when `running && !paused`, and runs `trackPerf`, which adapts a `quality` level (0–2) to keep frame rate up.
- **Input** — keyboard (`keys` map; `KeyP`/`Escape` pause, `KeyX`/`Shift` bomb, `Space` fire), mouse on the canvas, and touch joystick/fire/bomb buttons. Pointer and touch handlers are bound near lines 760–865.

### Conventions

- Keep everything in `index.html` unless the user asks for a split. Don't introduce a build step or dependencies without asking.
- Keep the single IIFE and `'use strict'`. Put new constants in the tuning tables above rather than hard-coding numbers in logic.
- Use the existing `localStorage` key prefix (`skyfighter.`) for any new persisted setting.
- Preserve accessibility hooks: the `srLive` live region is used by `announce()`, and reduced-motion is respected via `body.reduced-motion` and `screenShake`/flash gating.
- Gate any new visual effect that causes motion or flashing on the `reducedMotion` flag.

## Git workflow

- Develop on the branch specified by the task (currently `claude/gifted-tesla-7ra8i8`). Don't push to `main` directly.
- Write clear commit messages describing the gameplay or UI change.
- Don't open a pull request unless explicitly asked.

## Known gaps

- `README.md` contains only a timestamp. If you are asked to document the project for users, expand it with the game description, controls, and how to run it.
