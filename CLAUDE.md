# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a small, dependency-free browser game: **Sky Fighter**, a Raiden-style vertical shoot-'em-up.
The entire game lives in one self-contained file, `index.html` (inline `<style>` and `<script>`, HTML5 `<canvas>` 2D rendering, no build step, no frameworks, no external assets).

Files:

- `index.html` — the whole game (markup, CSS, and JavaScript).
- `README.md` — minimal placeholder; it is currently just the title.
- `CLAUDE.md` — this file.

There is no `package.json`, no test suite, no linter config, and no CI.

## Running and testing

- Run it by opening `index.html` directly in a browser, or serve the directory with any static server (e.g. `python3 -m http.server` and visit `http://localhost:8000`).
- No install step is required.
- There are no automated tests. Verify changes by loading the game in a browser and playing through the affected behavior (start, shoot, bomb, pause, boss fight, game over, difficulty change, mute, reduced-motion toggle, and touch controls where relevant).

## Architecture of `index.html`

The script is a single IIFE (`(function () { 'use strict'; ... })();`) so nothing leaks to `window`. Keep new code inside that scope.

Main sections, in file order (approximate line numbers):

- **Markup and CSS** (~lines 1–421): HUD (`#hud`, score/best/multiplier, hearts, bombs, weapon label), overlays (`#overlay`, start/pause/game-over), touch controls (`#touchControls`, joystick, fire and bomb buttons), and the `#gameCanvas` element.
- **Config constants** (~lines 463–528):
  - `STORAGE_*` keys for `localStorage` (best score, mute, difficulty, reduced motion). Keys are prefixed `skyfighter.`.
  - `DIFFICULTIES` (`easy` / `normal` / `hard`): spawn rate, enemy speed, enemy shot chance, starting lives, starting bombs, boss HP multiplier.
  - `KILLS_PER_STAGE` (18) controls stage progression.
  - `THEMES`: background palettes cycled per stage.
  - `ENEMY_TYPES` and `BOSSES`: enemy and boss definitions.
- **Audio** (~lines 545–680): lazily created `AudioContext` (`ensureAudio`), with `beep` and `noiseBurst` helpers and `EXPLODE_SFX` mappings. Audio starts only after a user gesture.
- **UI helpers** (~lines 680–752): HUD rendering, flashes, combo pop-ups, stage banner, screen shake, mute and difficulty setters, and `announce` for screen readers (`#srLive`).
- **Canvas sizing and input** (~lines 753–870): `BASE_W = 480`, `BASE_H = 720` logical resolution scaled by `resize()`. Virtual joystick, touch and mouse handling, and `canvasPointFromEvent`.
- **Game state and logic** (~lines 871–1506):
  - `resetGame()` initializes state.
  - `update(dt)` is the per-frame simulation: player movement, firing, enemies, bullets, collisions, power-ups, bosses.
  - `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`, `applyPowerUp`, `useBomb`, `fireWeapon`.
  - `BOSS_PATTERNS` is an array of bullet-pattern functions (`fireRadial`, `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`, `fireCross`, `fireHomingSeeded`, `fireDoubleRing`) used by `updateBoss`.
  - `loseLife`, `killBoss`, `endGame`, `togglePause`.
- **Rendering** (~lines 1507–1639): `drawPlane`, `drawBossShip`, `drawClouds`, and `draw()`. Drawing is done in a fixed logical 480×720 space.
- **Main loop** (~lines 1640–end): `loop(now)` uses `requestAnimationFrame`, clamps `dt` to 0.05 s, and only calls `update`/`draw` while running and not paused.

### Conventions

- Time-based movement: multiply by `dt` (seconds). Do not tie motion to frame count.
- Logical coordinates: game objects use the 480×720 base space. Do not use CSS pixel sizes inside game logic.
- Input: keyboard uses `KeyboardEvent.code` (`ArrowKeys`/`WASD` to move, `Space` to fire, `KeyX`/`Shift` to bomb, `KeyP`/`Escape` to pause). Touch and mouse map onto the same `keys`/`firing` state.
- Persistence: read and write `localStorage` only through the `STORAGE_*` constants. Wrap storage access in a try/catch if you add new calls, because storage can be blocked in some browsers.
- Accessibility: respect the reduced-motion setting (`reducedMotion`) for screen shake and flashes, and announce important state changes through `announce()`.

## Making changes

- Keep everything in `index.html` unless there is a clear reason to split files. If you do split, update this document.
- Do not introduce a build system, package manager, or external CDN dependencies without asking first. The project is intentionally a single static file.
- Preserve the single-IIFE structure and `'use strict'`.
- When tuning gameplay, change the `DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, or `BOSS_PATTERNS` constants rather than hard-coding numbers in logic.
- Keep the layout working on phones (the page uses `viewport-fit=cover`, safe-area insets, and `touch-action: none`). Check touch controls after any HUD or CSS change.

## Git

- Development branch for this repo: `claude/gifted-tesla-y4os30`.
- Commit messages should be short and imperative, describing the gameplay or code change.
- Do not open pull requests unless explicitly asked.
