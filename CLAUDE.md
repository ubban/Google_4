# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo contains **Sky Fighter**, a Raiden-style vertical shoot-'em-up written as a single self-contained HTML5 canvas game.

- `index.html` — the entire game: markup, CSS (`<style>`, ~lines 10–360), DOM (`<body>`, ~lines 361–421), and JavaScript (`<script>`, ~lines 422–end). There is no build step, no bundler, no dependencies, and no package manager.
- `README.md` — minimal placeholder. Not a source of truth for the game.

There are no tests, linters, or CI configuration in the repo. Verify changes by opening `index.html` in a browser (or serving the folder with any static server) and playing.

## Running the game

```bash
# any static server works; examples:
python3 -m http.server 8000     # then open http://localhost:8000
# or simply open index.html directly in a browser
```

## Code layout inside `index.html`

The script is one IIFE (`(function () { 'use strict'; ... })();`). Everything is private to that closure; do not add globals.

Major sections, in file order (approximate line numbers):

| Area | Key identifiers |
| --- | --- |
| DOM refs & constants | `canvas`, `ctx`, `overlay`, `STORAGE_*` keys (`skyfighter.*` in localStorage) |
| Adaptive quality | `quality`, `trackPerf()` — drops visual quality when frame time is high |
| Tuning tables | `DIFFICULTIES` (easy/normal/hard), `KILLS_PER_STAGE`, `THEMES` (per-stage sky palettes), `ENEMY_TYPES` (grunt/interceptor/tank), `BOSSES` (6 bosses) |
| Audio | `ensureAudio()`, `beep()`, `noiseBurst()`, `sfx` (WebAudio, procedurally generated — no audio files) |
| HUD / feedback | `renderHearts`, `renderBombs`, `flashHit`, `showComboPop`, `showStageBanner`, `screenShake` |
| Canvas sizing | `BASE_W = 480`, `BASE_H = 720`, `resize()` — logical resolution is fixed; CSS scales it |
| Input | `keys` (keyboard), mouse/pointer steering, `joystickStart/Move/End` (touch), fire/bomb touch buttons |
| Game state | `resetGame()`; state vars `player`, `bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`, `boss`, `score`, `lives`, `running`, `paused`, `gameOver` |
| Spawning | `spawnEnemy()`, `spawnBoss()`, `maybeSpawnPowerUp()` (power-up types: `life`, `bomb`, `weapon`) |
| Combat | `fireWeapon()` (Vulcan/Laser, 3 power levels), `useBomb()`, `addScore()` / `breakChain()` (kill-chain multiplier) |
| Boss | `BOSS_PATTERNS` (radial, aimed spread, columns, spiral, cross, homing, double ring), `updateBoss()`, `killBoss()` |
| Main loop | `update(dt)` (logic), `draw()` (rendering), `loop(now)` (rAF driver; `dt` clamped to 50 ms) |

Game flow: start button → `resetGame()` → `running = true` → `loop` calls `update` and `draw` each frame until `loseLife()` runs out of lives → `endGame()` shows the overlay with score/best/stage. Pause is toggled via `togglePause()`.

## Controls (must stay in sync with the on-screen hint)

- Mouse: move to steer, click to fire.
- Keyboard: Arrows or WASD to move, Space to fire, X or Shift for bomb, P or Esc to pause.
- Touch: drag in the left zone to fly (joystick), FIRE button, BOMB button. Touch UI is shown only when `isTouchDevice` is true.

If you change a binding, update the `#controlsHint` text in the DOM as well.

## Persistence

Stored in `localStorage` under the `skyfighter.*` keys:

- `skyfighter.bestScore` — high score
- `skyfighter.muted` — audio mute flag (`'1'` = muted)
- `skyfighter.difficulty` — `easy` | `normal` | `hard`
- `skyfighter.reducedMotion` — overrides the `prefers-reduced-motion` media query when set

Reads and writes should stay tolerant of storage being unavailable.

## Conventions

- **Single file.** Keep all code, styles, and markup in `index.html` unless the user asks for a restructure. Do not introduce a build system, framework, or npm dependencies without being asked.
- **No external assets.** Audio is synthesized with WebAudio and graphics are drawn with canvas primitives. Do not add image or sound files unless requested.
- **Keep the closure intact.** New code belongs inside the existing IIFE and should use `const`/`let`, matching the surrounding style (2-space indent, single quotes, semicolons).
- **Tuning lives in tables.** Prefer adjusting `DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `THEMES`, and `BOSS_PATTERNS` over hard-coding magic numbers in logic.
- **Respect the fixed logical resolution.** Game coordinates are in the 480×720 space; do not key positions off the CSS size.
- **Accessibility.** Announcements go through `announce()` (ARIA live region `#srLive`). Honor `reducedMotion` for screen shake and similar effects.
- **Frame-rate independence.** Movement and timers use `dt` (seconds). New timed behavior should do the same.

## Making changes

1. Make edits in `index.html` (or `README.md` for docs).
2. Manually test the affected flow in a browser: start, fire, bomb, pause/resume, power-ups, a boss fight, game over, and the difficulty / mute / reduced-motion toggles. Check both mouse/keyboard and touch layouts if you touched input or HUD code.
3. Confirm that the browser console shows no errors.
4. Commit with a clear, descriptive message and push to the branch you were asked to use.

## Git

- Develop on the branch specified by the task (currently `claude/gifted-tesla-lluelv`). Do not push to other branches without explicit permission.
- Do not open pull requests unless explicitly asked.
