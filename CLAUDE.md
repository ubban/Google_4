# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo is **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs in the browser. It is a
single self-contained file:

- `index.html` (~1,670 lines) — all markup, CSS (`<style>`, ~lines 10–362), and JavaScript
  (`<script>`, starting ~line 422). There is no build step, bundler, package manager, or test suite.
- `README.md` — a placeholder; no setup instructions.

There are no external dependencies. Rendering uses a 2D canvas; audio uses the Web Audio API.

## Running the game

No build is needed. Open `index.html` directly in a browser, or serve the directory with any static
server (e.g. `python3 -m http.server`) and visit it. Verify changes by playing in a browser, since
there are no automated tests.

## Code layout inside `index.html`

The script is one IIFE (`(function () { 'use strict'; ... })();`). Within it, the code is grouped by
the comment banners (`// ---------- Name ----------`):

- **DOM references** — `document.getElementById(...)` lookups at the top of the script. IDs must
  match the markup (e.g. `gameCanvas`, `overlay`, `startBtn`, `hearts`, `bombs`, `bossBarFill`,
  `touchControls`, `fireBtn`, `bombBtn`). Keep these in sync when editing HTML.
- **Storage keys** — `STORAGE_*` constants (`skyfighter.bestScore`, `skyfighter.muted`,
  `skyfighter.difficulty`, `skyfighter.reducedMotion`) used for `localStorage` persistence.
- **Adaptive performance** — `trackPerf` samples frame times and steps a `quality` level (0–2) up or
  down to reduce effects on slow devices.
- **Tuning tables** (edit these to rebalance; prefer changing data over logic):
  - `DIFFICULTIES` — `easy` / `normal` / `hard`: spawn rates, enemy speed, shoot chance, lives,
    bombs, and `bossHpMul`.
  - `KILLS_PER_STAGE` — kills needed to advance a stage (18).
  - `THEMES` — six background palettes cycled per stage.
  - `ENEMY_TYPES` — `grunt`, `interceptor` (weaving), `tank` (4 HP, stage 2+): size, HP, speed, colors,
    score, `minStage`.
  - `BOSSES` — six bosses (VANGUARD → TITAN) with colors and `hpMul`.
  - `BOSS_PATTERNS` — bullet patterns (radial, aimed spread, columns, spiral, cross, homing, double ring).
- **Audio** — `ensureAudio`, `beep`, `noiseBurst`, and the `sfx` helpers. Audio is created lazily on
  the first user gesture (browser autoplay policy); keep it that way.
- **Input** — keyboard (`keys` map), touch joystick + buttons, and mouse on the canvas.
- **Game state & logic** — `resetGame`, `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`,
  `applyPowerUp`, `fireWeapon`, `useBomb`, `addScore` / combo chain (`breakChain`), `loseLife`,
  `killBoss`, `endGame`, `togglePause`. `update(dt)` advances the simulation.
- **Rendering** — `draw()` and helpers (`drawPlane`, `drawBossShip`, `drawClouds`, `spawnExplosion`,
  `screenShake`). `draw()` runs only when the game is running and not paused.
- **Main loop** — `loop(now)` clamps `dt` to 50 ms, then calls `update` and `draw` each
  `requestAnimationFrame`.

### Controls

| Action        | Keyboard                       | Touch / mouse              |
|---------------|--------------------------------|----------------------------|
| Move          | Arrows or WASD                 | Left joystick / mouse move |
| Fire          | Hold Space                     | Fire button / mouse down   |
| Bomb          | `X` or Shift                   | Bomb button                |
| Pause         | `P` or `Esc`                   | Pause button               |
| Mute / motion | —                              | Top-bar buttons            |

## Conventions

- **Single file, no dependencies.** Do not add a build system, frameworks, or CDN scripts unless the
  user asks. Keep changes inside `index.html` unless there is a clear reason to split files.
- **Keep the IIFE and `'use strict'`.** Do not leak globals.
- **Time-based movement.** Game logic uses `dt` (seconds). Multiply per-frame speeds by `dt` rather
  than assuming a fixed frame rate.
- **Respect reduced motion.** `reducedMotion` toggles the `reduced-motion` body class and should
  gate screen shake and heavy effects.
- **Accessibility.** Announcements go through `announce()` into the `srLive` live region. Keep buttons
  as real `<button>` elements with labels.
- **Responsive canvas.** The logical size is `BASE_W` × `BASE_H` (480 × 720), scaled by `resize()`.
  Keep layout working on both desktop and touch devices.
- **Persist only via the `STORAGE_*` keys.** Wrap `localStorage` access as the existing code does.

## Making changes

1. Work on the designated feature branch; do not push to other branches without permission.
2. Edit `index.html` directly. Prefer changing the tuning tables over hard-coding numbers.
3. Test in a browser: start a game, check all three difficulties, reach a boss (after 18 kills in a stage),
   use a bomb, pause/resume, and confirm the best score persists after reload.
4. Check the console for errors, and test touch controls via device emulation when touching input code.
   (Boss spawning is triggered in `update()` once `stageKills >= KILLS_PER_STAGE` and no boss is warned yet.)
5. Commit with a clear message describing the gameplay or code change.

## Things to watch out for

- The HUD and overlay rely on hidden/visible toggles (`hidden` attribute); keep them consistent when
  adding new screens or indicators.
- `togglePause()` is a no-op unless a game is running and not over — respect that guard.
- Audio must start from a user gesture; calling `ensureAudio()` from input handlers is intentional.
- The `dt` clamp (50 ms) prevents huge jumps after tab switches; do not remove it.
