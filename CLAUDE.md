# CLAUDE.md

Guidance for Claude Code when working in this repository.

## What this repo is

Sky Fighter, a single-file, Raiden-style vertical shoot-'em-up that runs in the browser on a `<canvas>`. There is no build step, no package manager, no dependencies, and no test suite.

Files:

- `index.html` — the entire game: CSS (`<style>`, roughly lines 10–421), markup, and one inline `<script>` (roughly lines 422–1668).
- `README.md` — a placeholder; no project docs live there yet.

## Running and verifying

- Open `index.html` directly in a browser, or serve the directory with any static server (e.g. `python3 -m http.server`).
- There are no tests, linters, or build commands. Verify changes by playing the game in a browser: start a run on each difficulty, check firing, bombs, pickups, a stage transition, a boss fight, pause/resume, and game over.
- Keep the file dependency-free. Do not add libraries, CDN scripts, or a build pipeline unless the user asks for one.

## Code layout inside `index.html`

The script is one IIFE: `(function () { ... })();`. Everything is private to it; the only globals are DOM elements looked up by id.

Roughly in order inside the script:

1. **DOM refs and storage keys** — `getElementById` lookups; `STORAGE_*` constants (`skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion`).
2. **Perf tracking** — `trackPerf` steps `quality` (0 full / 1 medium / 2 low) down when the average frame time over 45 frames is slow, and back up when it is fast.
3. **Tuning tables** — `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES` (background palettes, cycled by stage), `ENEMY_TYPES` (`grunt`, `interceptor`, `tank`), `BOSSES` (cycled by stage).
4. **Audio** — lazy `ensureAudio()` builds the `AudioContext`; `beep`, `noiseBurst`, and the `sfx` table. Audio must start from a user gesture, so call `ensureAudio()` in input handlers.
5. **HUD and effects** — `renderHearts`, `renderBombs`, flashes, combo pop, stage banner, screen shake.
6. **Difficulty and resize** — `setDifficulty`, `resize()` (fits a 480×720 logical canvas, scaled up to 1.6×).
7. **Input** — keyboard (`keys` map by `e.code`), touch joystick and FIRE/BOMB buttons, and mouse steering/firing on the canvas.
8. **Game state** — `resetGame()` sets all mutable state (`player`, `bullets`, `enemies`, `enemyBullets`, `powerUps`, `boss`, `score`, `lives`, `bombCount`, `stage`, `stageKills`, …).
9. **Drawing** — `drawPlane`, `drawBossShip`, `drawClouds`, and `draw()`.
10. **Simulation** — `update(dt)` (player movement, firing, spawning, collisions, power-ups), `spawnEnemy`, `spawnBoss`, `useBomb`, `fireWeapon`, boss attack patterns (`fireRadial`, `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`, `fireCross`, `fireHomingSeeded`, `fireDoubleRing`) driven by `updateBoss(dt)`, and `killBoss()`.
11. **Loop** — `loop(now)` clamps `dt` to 0.05 s, then calls `update` and `draw` only while `running && !paused`.

Game flow: each stage spawns enemies until `stageKills >= KILLS_PER_STAGE`, then a boss spawns. Defeating the boss calls `killBoss()`, which increments `stage`, resets `stageKills`, and shows the next stage banner. Lives, bombs, and weapon state carry over between stages.

## Controls (keep the hint text in sync)

- Keyboard: arrows or WASD move; Space fires (also held fire); X or Shift uses a bomb; P or Esc pauses.
- Mouse: move to steer, click to fire.
- Touch: drag left to fly, tap FIRE, tap BOMB.

The on-screen hint is `#controlsHint` in the markup. Update it whenever a control changes.

## Gameplay rules worth knowing

- Difficulty (`easy` / `normal` / `hard`) sets lives, bombs, spawn rate, enemy speed, enemy fire chance, and boss HP multiplier. Read them from `DIFFICULTIES`; do not hard-code them.
- Power-ups: `life` (up to `maxLives`), `bomb` (up to `maxBombs`), and `weapon`. The weapon power-up raises the level 1→3 for the current weapon, then swaps between `vulcan` and `laser` and resets to level 1.
- Bombs clear enemy bullets and enemies and deal 24 damage to the boss.
- Best score is saved to `localStorage` under `skyfighter.bestScore`. Storage calls are not wrapped in try/catch; if you add more, guard them, since private browsing can throw.
- Reduced motion (`reducedMotion`) disables screen shake, flashes, and pulses. It defaults from `prefers-reduced-motion` until the user toggles it.

## Conventions

- Match the existing style: 2-space indentation, single quotes in JS, short comments only where the logic is not obvious, and compact one-line object literals for tuning data.
- Put new tuning values in the existing tables (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `THEMES`) rather than as magic numbers in the logic.
- Keep the game working without a network connection and on touch devices. Test any change that touches input or layout at phone width.
- Preserve accessibility hooks: `srLive` announcements via `announce()`, and the `title`/labels on icon buttons.

## Git

- Development branch for this task: `claude/gifted-tesla-55378u`. Push with `git push -u origin <branch>`.
- Commit messages are short, imperative summaries of the change (see `git log`).
- Do not open pull requests unless the user asks.
