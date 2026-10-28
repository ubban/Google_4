# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Google_4 is a small, dependency-free browser game, **Sky Fighter**: a Raiden-style vertical shoot-em-up rendered on a `<canvas>`. The entire game is a single file, `index.html`, containing inline CSS and one inline `<script>` IIFE. `README.md` is a placeholder with only the project title.

There is no build step, package manager, bundler, test suite, or linter. To run the game, open `index.html` in a browser.

## Repository layout

| Path | Purpose |
| --- | --- |
| `index.html` | The whole game: markup (HUD, touch controls, start overlay), CSS (`<style>` block), and game logic (`<script>` block). |
| `README.md` | Placeholder. Update it if the project gains real documentation. |

## Architecture of `index.html`

The script runs inside a single `(function () { 'use strict'; ... })()` wrapper, so nothing leaks to `window`. Keep new code inside that wrapper.

Major sections, in the order they appear in the script:

1. **DOM references**: `const` lookups for every element by id, grouped at the top.
2. **Storage keys**: `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion`. These are the only `localStorage` keys. Reads and writes are direct, with no try/catch.
3. **Tuning tables**:
   - `DIFFICULTIES` (`easy` / `normal` / `hard`): spawn timing, enemy speed, enemy fire chance, starting lives and bombs, and boss HP multiplier.
   - `KILLS_PER_STAGE` (18): kills needed before a boss warning.
   - `THEMES`: six sky palettes, cycled by stage.
   - `ENEMY_TYPES`: `grunt`, `interceptor` (weaving), and `tank` (4 HP, from stage 2).
   - `BOSSES`: six boss designs, cycled by stage, each with a `hpMul`.
   - `BOSS_PATTERNS`: bullet patterns per boss design.
4. **Audio**: a lazily created `AudioContext` (`ensureAudio`) with `beep` and `noiseBurst` primitives. Named effects live in the `sfx` object. Sound is muted via `muted`.
5. **Rendering helpers**: `drawPlane`, `drawBossShip`, `drawClouds`, and `draw()`. The render path scales with `quality` (0 full, 1 medium, 2 low), which `trackPerf` adjusts from frame times.
6. **Input**: keyboard (`keys` map keyed by `e.code`), mouse steering on the canvas, and touch (virtual joystick in `moveZone`, plus `fireBtn` and `bombBtn`).
7. **Game state**: module-level `let` variables (`player`, `bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`, `boss`, `score`, `lives`, `stage`, and so on). `resetGame()` initializes them for a new run.
8. **Simulation**: `update(dt)` advances spawning, movement, collisions, and boss logic. `updateBoss(dt)` drives boss phases and patterns.
9. **Game flow**: `loseLife()`, `endGame()`, `killBoss()`, `togglePause()`.
10. **Loop**: `loop(now)` computes `dt`, calls `update` and `draw`, and reschedules with `requestAnimationFrame`. It starts when the script loads, and it stays idle until `startBtn`'s click handler calls `resetGame()` to begin a run.

### Controls

- Desktop: mouse moves the ship and click fires. Arrow keys or WASD move. Space fires. X or Shift triggers a bomb. P or Escape pauses.
- Touch: drag left to fly, tap FIRE, tap BOMB.

### Stage progression

Each stage spawns enemies until `KILLS_PER_STAGE` kills, then warns of a boss. Defeating the boss increments `stage`, which changes the theme, boss design, and enemy mix. Boss HP scales with `stage` and the boss's `hpMul`.

## Conventions

- **Single file**: keep the game in `index.html` unless the user asks for a split. Do not add a build step or dependencies without asking.
- **Style**: match the existing code. It uses 2-space indentation, single quotes in JS, `const` by default, arrow functions for callbacks, and short helper functions. Keep comments sparse and only where they explain something non-obvious, as the existing code does.
- **Tuning changes**: prefer editing the tables (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `THEMES`) over hard-coding numbers in logic.
- **Accessibility**: the game has a reduced-motion toggle (`reducedMotion`, which also defaults from `prefers-reduced-motion`) that gates screen shake, flashes, and pulses. New visual effects that shake or flash should respect it. Screen-reader announcements go through `announce()` into the `#srLive` region.
- **Responsive scaling**: `resize()` scales the fixed `BASE_W` × `BASE_H` (480 × 720) canvas to fit the window, capped at 1.6×. Layout and touch targets must work at phone width.

## Verifying changes

There is no automated test or lint setup. To verify a change:

1. Open `index.html` in a browser (a local static server also works).
2. Start a run and check the affected behavior: movement, firing, bombs, pausing, stage clear and boss fight, game over, and the best-score save.
3. Check the touch controls with device emulation, and check the reduced-motion and mute toggles if you touched effects or audio.
4. Check the browser console for errors.

Syntax errors in the inline script stop the whole game, so load the page after every script edit.

## Git

- Development branch for this work: `claude/gifted-tesla-9pckss`.
- Commit messages should be short and descriptive, in the imperative mood, as in the existing history (for example, "Add Sky Fighter: Raiden-style vertical shoot-em-up game").
