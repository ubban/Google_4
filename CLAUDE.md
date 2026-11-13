# CLAUDE.md

Guidance for Claude Code when working in this repository.

## What this repo is

`Google_4` is a small, dependency-free browser game, **Sky Fighter**: a Raiden-style vertical shoot-'em-up. The entire game lives in one file, `index.html` (inline `<style>` and `<script>`, drawn on a `<canvas>`). `README.md` is a placeholder.

There is no build step, package manager, test suite, or linter configured.

## Running and testing

- Open `index.html` directly in a browser, or serve the directory with any static server (e.g. `python3 -m http.server`).
- There are no automated tests. Verify changes by playing the game in a browser, on both desktop (keyboard/mouse) and a touch/mobile viewport.
- Use a browser or Playwright to check the game still starts, the HUD renders, and the stage/boss flow works after non-trivial edits.

## Code layout (`index.html`)

Roughly in file order:

1. **`<head>`**: meta tags, an inline SVG favicon, and the full stylesheet. HUD, hearts/bombs, vignette, and overlay styles are all here.
2. **`<body>`**: the `#container`, `#gameCanvas`, HUD elements, touch controls (joystick `moveZone`, `fireBtn`, bomb button), and the start/pause/game-over overlays. The `#controlsHint` text lists the controls.
3. **Config constants** (near line ~463):
   - `STORAGE_*` keys (all `localStorage`, prefixed `skyfighter.`): best score, mute, difficulty, reduced motion.
   - `DIFFICULTIES` (`easy` / `normal` / `hard`): spawn rates, enemy speed, shoot chance, starting lives and bombs, boss HP multiplier.
   - `KILLS_PER_STAGE` (18), `THEMES` (sky palettes cycled per stage), `ENEMY_TYPES` (`grunt`, `interceptor`, `tank`), `BOSSES` (cycled by stage).
4. **Audio**: lazily created `AudioContext` (`ensureAudio`), with `beep` and `noiseBurst` helpers. Audio is created only after a user gesture.
5. **HUD and effects**: `renderHearts`, `renderBombs`, `flashHit`, `showStageBanner`, `screenShake`, combo popups.
6. **Input**: keyboard (`keys` map), touch joystick, and mouse/pointer handling. Listeners are registered near lines ~769–832.
7. **Game state and loop**: `let` state variables (score, lives, bombs, stage, chain/multiplier, etc.), `resetGame`, `update(dt)`, and the draw functions (`drawPlane`, `drawBossShip`). Performance is tracked by `trackPerf`, which adjusts a `quality` level (full / medium / low).
8. **Spawning and combat**: `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`, `applyPowerUp`, `fireWeapon`, `useBomb`, and the boss bullet-pattern helpers (`fireRadial`, `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`, `fireCross`, `fireHomingSeeded`, `fireDoubleRing`).

Line numbers drift as the file changes; search by identifier rather than trusting numbers.

## Controls (keep the hint text in sync)

- Keyboard: Arrows/WASD move, Space fire, X or Shift bomb, P or Esc pause.
- Mouse: move to steer, click to fire.
- Touch: drag left side to fly, tap FIRE, tap BOMB.

If you add or change a control, update the `#controlsHint` text and the matching key handlers together.

## Conventions

- **Keep it a single file** unless the user asks for a split. Don't add a bundler, framework, or npm dependencies without asking.
- **Vanilla JS, no modules.** Match the existing style: `const`/`let`, arrow functions, and two-space indentation in the `<script>` and `<style>` blocks.
- **Use the existing tuning tables.** Balance changes should go into `DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, and `THEMES` rather than being hardcoded in logic.
- **Respect the user preferences** persisted in `localStorage` (mute, difficulty, reduced motion). Any new setting should follow the same `STORAGE_*` pattern and wrap storage in a way that still works if storage is unavailable.
- **Respect reduced motion and performance tiers.** New visual effects should honor `reducedMotion` and the `quality` level, and should not add per-frame allocations in the hot path without need.
- **Mobile first.** The layout must work at phone width with touch controls; `touch-action: none` and `user-select: none` are intentional on the page.
- **Audio** starts only after a user gesture. Don't call `ensureAudio` or create an `AudioContext` on page load.

## Git

- Development branch for this work: `claude/gifted-tesla-apog17`. Commit and push there; don't push to other branches without permission.
- Don't open a pull request unless the user explicitly asks.
