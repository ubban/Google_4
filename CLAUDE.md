# CLAUDE.md

Guidance for Claude Code when working in this repository.

## What this repo is

A small static web game, **Sky Fighter**: a Raiden-style vertical shoot-'em-up. The entire game lives in one self-contained file, `index.html` (about 1,670 lines: inline CSS, markup, and one inline `<script>`). There is no build step, no package manager, no bundler, and no test suite.

Files:

- `index.html`: the whole game (markup, styles, and game logic).
- `README.md`: a placeholder. It only contains the title and a stray timestamp line.

## Running it

Open `index.html` directly in a browser, or serve the directory with any static server, for example `python3 -m http.server`. No install step is needed.

## Code layout inside `index.html`

- **Styles** (`<style>`, roughly lines 10–359): CSS variables are not used. Colors are inline. The HUD, touch controls, overlay, and the reduced-motion and vignette/flash effects are all styled here.
- **Markup** (roughly lines 360–421): `#container` holds the `#gameCanvas` canvas plus HUD (`#hud`), boss bar, stage banner, top bar buttons (`#muteBtn`, `#motionBtn`, `#pauseBtn`), touch controls, and the start/title overlay (`#overlay`, `#startBtn`, `.diffBtn` difficulty buttons).
- **Script** (roughly lines 421–1666, one IIFE-style block): everything runs inside it. Key sections, in file order:
  - DOM references and storage keys (`STORAGE_BEST`, `STORAGE_MUTE`, `STORAGE_DIFF`, `STORAGE_REDUCED`).
  - Tuning tables: `DIFFICULTIES` (easy / normal / hard), `KILLS_PER_STAGE`, `THEMES` (per-stage sky palettes), `ENEMY_TYPES` (grunt, interceptor, tank), and `BOSSES` (six boss designs, cycled by stage).
  - Performance: `trackPerf` adapts a `quality` level (0 full, 1 medium, 2 low) from average frame time. Particle counts scale with it.
  - Audio: a lazily created `AudioContext` (`ensureAudio`), with `beep` and `noiseBurst` primitives and a named `sfx` table. Audio is only created after a user gesture.
  - Input: keyboard (`keys` map, keyed by `e.code`), mouse steering and click-to-fire on the canvas, and touch (virtual joystick in `#moveZone`, `#fireBtn`, `#bombBtn`).
  - Rendering helpers: `drawPlane`, `drawBossShip`, `draw()`, `drawClouds`. `resize()` scales the canvas to fit, with a fixed logical size of `BASE_W` × `BASE_H` = 480 × 720.
  - Game state: a set of top-level `let`s (`player`, `bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`, `boss`, `score`, `lives`, `bombCount`, `stage`, `chain`/`multiplier`, and so on). `resetGame()` initializes them.
  - Simulation: `update(dt)` drives spawning, collisions, and power-ups. `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`, `fireWeapon`, and `useBomb` live here. `updateBoss(dt)` runs boss phases and telegraphs, and the `fire*` helpers (`fireRadial`, `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`, `fireCross`, `fireHomingSeeded`, `fireDoubleRing`) produce boss bullet patterns. `BOSS_PATTERNS` maps each boss index to its patterns.
  - Flow: `endGame()` shows the game-over overlay and writes the best score. Stage progression happens when `KILLS_PER_STAGE` is reached and the boss is defeated.
  - Main loop: `loop(now)` runs with `requestAnimationFrame`.

## Gameplay facts to keep consistent

- Difficulty changes lives, starting bombs, spawn rate, enemy shooting chance, and boss HP multiplier (`DIFFICULTIES`).
- Enemies are gated by stage through `minStage` (`ENEMY_TYPES`). Tanks appear from stage 2.
- Weapons: `player.weapon` is `'vulcan'` or `'laser'`, with a `level`. The HUD shows the name via `weaponLabel()`.
- Scoring uses a chain: `addScore` raises `chain`, and `multiplier` grows with it (capped at 10×). The chain resets after about 2.2 s without a kill.
- Power-up types dropped by enemies: `life`, `bomb`, `weapon`.
- Stage themes cycle through `THEMES` by `(stage - 1) % THEMES.length`. Bosses cycle through `BOSSES` the same way.

## Persistence

`localStorage` keys, all prefixed `skyfighter.`:

- `skyfighter.bestScore`
- `skyfighter.muted`
- `skyfighter.difficulty`
- `skyfighter.reducedMotion`

Reads and writes are not wrapped in try/catch. Keep that in mind if you add storage calls, since private browsing or blocked storage can throw.

## Conventions and guidance

- Keep the game as a single file unless the user asks for a restructure. Don't add a build step, framework, or dependencies without a clear request.
- Prefer small, targeted edits. Don't reformat unrelated code.
- Preserve the accessibility features: `#srLive` announcements via `announce()`, the reduced-motion toggle (`reducedMotion`, which gates screen shake and flash), and the `prefers-reduced-motion` default.
- Preserve touch support. Controls must keep working on phones (`touch-action: none`, passive: false listeners where `preventDefault` is needed).
- Tuning numbers (spawn rates, HP, speeds, `DIFFICULTIES`) are deliberate balance values. Change them only when the task calls for balance work.
- Verify changes by loading the page in a browser and playing a few stages, including a boss fight, on both desktop and a narrow mobile viewport. There are no automated tests.

## Git

- Development branch for this session: `claude/gifted-tesla-9evzc0`.
- Commit messages and PRs should describe the change. Don't open a PR unless asked.
