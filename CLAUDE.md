# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo contains **Sky Fighter**, a Raiden-style vertical shoot-em-up game built as a single self-contained HTML5 canvas page.

- `index.html` — the entire game: markup, CSS (`<style>`, ~lines 10–362), canvas element, and one inline `<script>` IIFE (~lines 422–1668). There is no build step, no bundler, no framework, and no external dependencies.
- `README.md` — minimal placeholder.

To run the game, open `index.html` in a browser (or serve the directory with any static server). No install is needed.

## Architecture of `index.html`

The script is one IIFE with `'use strict'`. Sections, in source order:

1. **DOM references** — every element is looked up once by id at the top of the script. Keep ids in sync between markup and script.
2. **Persistence keys** — `localStorage` keys prefixed `skyfighter.` (`bestScore`, `muted`, `difficulty`, `reducedMotion`). All storage access is wrapped by the game; preserve the prefix when adding keys.
3. **Adaptive quality** — `trackPerf()` samples frame times and steps `quality` (0 full / 1 medium / 2 low) to keep frame rate up.
4. **Tuning data** — declarative constants:
   - `DIFFICULTIES` — spawn rates, enemy speed/shoot chance, starting `lives`, `bombs`, `bossHpMul`.
   - `KILLS_PER_STAGE` — kills needed before a stage advances.
   - `THEMES` — per-stage sky gradient, cloud/sun colors, and name.
   - `ENEMY_TYPES` — `grunt`, `interceptor` (weaving), `tank` (4 HP, unlocks at stage 2).
   - `BOSSES` — six boss definitions; `hpMul` scales HP.
5. **Audio** — `ensureAudio()` lazily creates the `AudioContext` (browsers require a user gesture); `beep()` and `noiseBurst()` synthesize all sound effects. There are no audio files.
6. **HUD helpers** — hearts, bombs, combo pops, stage banner, screen shake, hit/bomb flash.
7. **Input** — keyboard (`keys` map keyed by `e.code`), touch joystick + FIRE/BOMB buttons, and mouse/pointer drag on the canvas (`canvasPointFromEvent`).
8. **Game state and actions** — `resetGame()`, `spawnEnemy()`, `spawnBoss()`, `maybeSpawnPowerUp()`, `addScore()` / `breakChain()` (kill-chain multiplier), `useBomb()`, `fireWeapon()`, `applyPowerUp()`, `loseLife()`, `endGame()`, `togglePause()`.
9. **Boss patterns** — `BOSS_PATTERNS` is an array of bullet-pattern functions (radial, aimed spread, columns, spiral, cross, homing, double ring). `updateBoss()` cycles through them.
10. **Update / draw** — `update(dt)` runs game logic with a delta-time step; `draw()` renders everything; `loop(now)` drives the `requestAnimationFrame` loop. Drawing is procedural (`drawPlane`, `drawBossShip`, `drawClouds`) — there are no image assets.
11. **Start** — the Start button calls `resetGame()` and hides the overlay; the loop is kicked off with `requestAnimationFrame(loop)` at the end.

Game logic runs on a fixed base canvas of **480×720**; `resize()` scales the displayed canvas to fit the window (capped at 1.6×) while keeping the internal resolution fixed. Write all coordinates and sizes in base-canvas units.

## Controls

| Action | Keyboard | Touch / mouse |
| --- | --- | --- |
| Move | Arrow keys or WASD | Left-side joystick, or drag on canvas |
| Fire | Space (hold) | FIRE button (hold) or mouse-down |
| Bomb | X or Shift | BOMB button |
| Pause | P or Escape | Pause button (top-right bar, with mute and reduced-motion toggles) |

## Gameplay facts to keep in mind

- **Weapons**: Vulcan and Laser, each with 3 power levels, upgraded by power-ups.
- **Bombs** are limited per difficulty; they clear the screen and damage bosses.
- **Lives** are shown as hearts; losing all lives ends the run.
- **Stages** are gated by `KILLS_PER_STAGE` kills. When the quota is reached a boss spawns (after a short delay); defeating it advances the stage, cycling through `THEMES` with `(stage - 1) % THEMES.length`. The boss health bar is `bossBarWrap`.
- **Scoring** uses a kill-chain multiplier that resets via `breakChain()`.

## Development conventions

- **Single file.** Keep the game in `index.html` unless the user asks for a split. Don't introduce a build system, package manager, or external CDN dependencies without asking.
- **No tests or linter are configured.** Verify changes by loading the page in a browser and playing through the affected path (start, shoot, bomb, pause, game over, and any stage/boss code you touched). A headless Chromium with Playwright is available in the cloud environment, and `/run` can drive the app.
- **Keep tuning in the constant tables.** Balance changes belong in `DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, and `THEMES`, not hard-coded inside `update()`/`draw()`.
- **Respect the time step.** Movement and timers use the `dt` passed to `update(dt)`; don't add frame-count-based motion.
- **Respect reduced motion.** Screen shake and flashes are gated by `reducedMotion`; new motion effects should honor it too.
- **Accessibility.** Status messages go through `announce()` (live region `srLive`). Buttons need text or a `title`.
- **Audio** must start from a user gesture. Call `ensureAudio()` from input handlers before playing sound.
- **Mobile.** Touch controls are shown on small screens; test any control change with touch events as well as keyboard.

## Git workflow

- Develop on the branch designated for the session (currently `claude/gifted-tesla-su45ni`). Don't push to other branches without explicit permission.
- Commit messages should describe the gameplay or code change in the imperative, with a short body for non-trivial work (the existing history uses that style).
- Don't open a pull request unless the user asks for one.
