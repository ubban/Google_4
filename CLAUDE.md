# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Google_4 is a static, dependency-free web project. Its only real content is **Sky Fighter**, a Raiden-style vertical shoot-'em-up written as a single-file HTML5 canvas game.

- `index.html` — the entire game: markup, CSS (`<style>`, ~lines 10–359), and JavaScript (`<script>`, ~lines 422–1667).
- `README.md` — a one-line title. No other docs.

There is no build step, package manager, bundler, test suite, or linter. Open `index.html` in a browser to run it.

## Working in this repo

- **Keep it a single file** unless the user asks for a split. Don't add a `package.json`, build tooling, or frameworks without an explicit request.
- **Verify changes in a browser.** No automated tests exist, so after editing game logic, load `index.html` (e.g. via `python3 -m http.server` and a headless browser) and check the affected behavior: start, fire, bomb, pause, game over, and stage transitions.
- Pre-installed Chromium/Playwright can be used for that check; no install is needed.
- Keep the existing style: 2-space indentation, single quotes in JS, `const`/`let`, and the `// ---------- Section ----------` banner comments that divide the script.

## Game architecture (`index.html`)

The script runs inside one IIFE with `'use strict'`. Major pieces, in file order:

1. **DOM refs and storage keys** — element lookups at the top; `STORAGE_*` constants prefixed `skyfighter.`.
2. **Adaptive performance** — `trackPerf` samples frame time and steps `quality` (0 full / 1 medium / 2 low) to lower effects cost.
3. **Tuning tables** — `DIFFICULTIES` (easy/normal/hard: spawn rate, enemy speed, shot chance, lives, bombs, boss HP multiplier), `KILLS_PER_STAGE` (18), `THEMES` (per-stage sky palettes), `ENEMY_TYPES` (grunt, interceptor, tank), `BOSSES` (six bosses, cycled by stage).
4. **Persistence** — best score, mute, difficulty, and reduced-motion are saved to `localStorage`. Reads are not wrapped in try/catch today; keep that in mind if you touch them.
5. **Audio** — a lazily created `AudioContext` (`ensureAudio`) with `beep` / `noiseBurst` helpers and an `sfx` object. Audio starts only after a user gesture.
6. **HUD helpers** — hearts, bombs, combo pops, stage banner, screen shake, flash, and the `announce` live region for screen readers.
7. **Input** — keyboard state in `keys`; touch joystick and buttons for mobile; mouse-drag on the canvas.
8. **Game state** — `resetGame()` initializes `player`, `bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`, `boss`, plus `score`, `lives`, `running`, `paused`, `gameOver`.
9. **Simulation** — `update(dt)` runs spawning, collisions, scoring (kill-chain multiplier via `addScore` / `breakChain`), stage progression, and power-ups. `fireWeapon()` implements Vulcan (3 levels of spread) and Laser; a weapon-swap power-up toggles between them. `useBomb()` clears the screen and damages bosses.
10. **Bosses** — `spawnBoss`, `updateBoss`, `killBoss`, and `BOSS_PATTERNS`, a set of bullet-pattern helpers (radial, aimed spread, columns, spiral, cross, homing, double ring).
11. **Rendering** — `draw()` paints everything, using `drawPlane`, `drawBossShip`, and `drawClouds` for parallax layers.
12. **Loop** — `loop(now)` clamps `dt` to 50 ms, updates and draws only when running and not paused, and calls `requestAnimationFrame`.

### Controls

- Move: arrow keys / WASD, mouse on canvas, or touch joystick.
- Fire: Space or the FIRE button (hold).
- Bomb: `X` or Shift, or the BOMB button.
- Pause: `P` or Escape.

## Conventions and gotchas

- **Game state is module-local.** Everything lives inside the IIFE; don't expose globals unless needed.
- **Frame-rate independence:** movement and timers use `dt` in seconds. Keep new motion and timers in that unit.
- **Reduced motion:** screen shake, flash, and pulse effects must check `reducedMotion` (the `reduced-motion` body class mirrors it for CSS).
- **Tuning changes** belong in the tables near the top of the script, not inline in the logic.
- **Git:** develop on the branch the session specifies (currently `claude/gifted-tesla-tupbj3`). Don't open a pull request unless the user asks.
