# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs in the browser. The entire game is a single self-contained file, `index.html` (~1,670 lines), with inline `<style>` and an inline `<script>`. There is no framework, no bundler, no package manager, and no external assets. Audio is synthesized with the Web Audio API.

Files:

- `index.html` — the whole game (markup, CSS, and JavaScript).
- `README.md` — minimal project readme.
- `CLAUDE.md` — this file.

## Running and testing

- **Run:** open `index.html` directly in a browser, or serve the directory with any static server (e.g. `python3 -m http.server`) and visit it.
- **Build:** none. Edit the file and reload.
- **Tests:** none exist. There is no test runner, linter, or CI config. Verify changes by playing the game in a browser (desktop keyboard and, if relevant, touch emulation). When you have a headless browser available, a quick Playwright smoke test (load page, click Start, confirm no console errors) is a reasonable check.

## Code layout inside `index.html`

The `<head>` holds meta tags (viewport locked to `user-scalable=no`, `theme-color` `#0b1d3a`) and the CSS. `<body>` contains a `<canvas id="gameCanvas">`, the HUD elements, the overlay/menu, and the touch controls. The script is one IIFE-style block at the bottom of the file, organized roughly as follows:

1. **DOM references and constants** — cached element handles (`scoreEl`, `overlay`, `touchControls`, …) and `STORAGE_*` localStorage keys.
2. **Tuning data** — `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES` (per-stage sky palettes), `ENEMY_TYPES` (grunt / interceptor / tank), and `BOSSES` (six boss designs, cycled by stage).
3. **Persistence and settings** — best score, mute, difficulty, and reduced-motion preference, all stored in localStorage.
4. **Audio** — lazily created `AudioContext` (`ensureAudio`), `beep` / `noiseBurst` primitives, and the `sfx` table of named sound effects.
5. **HUD and feedback** — hearts/bombs rendering, combo pop, stage banner, screen shake, hit/bomb flashes.
6. **Stage scaling** — `resize()` fits a fixed 480×720 logical canvas into the window (max scale 1.6).
7. **Input** — keyboard (`keys` map keyed by `e.code`), mouse/pointer target, and a virtual joystick plus fire/bomb buttons for touch.
8. **Game state** — mutable globals (`player`, `bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`, `boss`, `score`, `lives`, `stage`, …) and `resetGame()`.
9. **Combat and spawning** — `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`, `fireWeapon`, `useBomb`, `applyPowerUp`.
10. **Boss patterns** — `BOSS_PATTERNS[designIndex][patternIndex]` are functions that emit enemy bullets (radial, aimed spread, columns, spiral, cross, homing, double ring). `updateBoss` / `killBoss` drive boss phases.
11. **Rendering** — `drawPlane`, `drawBossShip`, `draw()`, and cloud/parallax helpers, all on the 2D canvas context.
12. **Main loop** — `update(dt)` then `draw()`, driven by `requestAnimationFrame` in `loop(now)`. `endGame()` handles game over and best-score saving.

## Key mechanics to keep in mind

- **Stages:** after `KILLS_PER_STAGE` (18) kills, the boss warning fires (`bossWarned`) and the stage's boss spawns. Enemy types unlock by `minStage`.
- **Weapons:** `vulcan` and `laser`, each with levels 1–3. Power-ups upgrade the level, and past level 3 they toggle the weapon type.
- **Bombs and lives:** bombs are triggered with X or Shift; lives and bomb counts are capped by `maxLives` / `maxBombs` and vary by difficulty.
- **Scoring:** base points come from `ENEMY_TYPES[...].score`, multiplied by a chain/combo `multiplier` that resets via `breakChain()`.
- **Adaptive quality:** `trackPerf` samples frame times and steps `quality` (0 full, 1 medium, 2 low) to reduce glow, shadows, and particle counts when frame rate drops.
- **Reduced motion:** honors `prefers-reduced-motion` on first load (unless the user has set a stored preference) and can be toggled from the UI. When on, hit/bomb flashes are skipped and screen shake is scaled down to ~15% and capped at 0.1s.

## Controls (reference)

- Move: arrow keys / WASD, or mouse/pointer (the ship follows the pointer), or the on-screen joystick on touch devices.
- Fire: Space (hold to keep firing), or the fire button on touch.
- Bomb: X or Shift, or the bomb button.
- Pause: P or Escape.

## Conventions

- **Single file:** keep everything in `index.html` unless the user asks to split it. Do not add a build step, package manager, or external CDN dependency without asking.
- **No dependencies:** use plain DOM and Canvas 2D / Web Audio APIs.
- **Style:** match the surrounding code — 2-space indentation, single quotes in JS, `const` by default, `let` for reassigned game state, arrow functions for callbacks, and terse helper names (`beep`, `fireRadial`, `rectsOverlap`).
- **Constants over magic numbers:** put tuning values in `DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, or the local constants block rather than scattering literals through the logic.
- **Frame-rate independence:** movement and timers are scaled by `dt` (seconds). Preserve this when adding entities or timers.
- **Logical resolution:** all game coordinates are in the 480×720 logical space; do not hard-code CSS pixel sizes in game logic.
- **Persistence keys:** new localStorage keys should use the `skyfighter.*` prefix and be wrapped in sensible defaults, since storage can be unavailable.
- **Audio:** sounds are created only after a user gesture (`ensureAudio`). Keep new effects routed through `sfx` and respect the `muted` flag.
- **Accessibility:** announcements go through `announce()` / `srLive`; keep the reduced-motion and mute toggles working when you add effects.

## Working in this repo

- Default branch is `main`. Feature work for this environment happens on the designated `claude/*` branch; do not push to other branches without permission.
- Keep commits focused, with descriptive messages.
- Because the game is one large file, prefer targeted `Edit` changes over rewriting whole sections, and re-read the affected function before editing.
- After changing gameplay, check the affected stage/boss/difficulty paths by playing, since there are no automated tests.
