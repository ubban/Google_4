# CLAUDE.md

Guidance for Claude Code when working in this repository.

## What this repo is

`Google_4` is a small, single-file web project containing **Sky Fighter**, a Raiden-style vertical shoot-'em-up built on an HTML5 `<canvas>`.

| File | Purpose |
| --- | --- |
| `index.html` | The entire game: inline `<style>` (~lines 10–359), markup (~361–421), and one inline `<script>` IIFE (~422–1667). |
| `README.md` | Placeholder project readme (currently just the title). |

There is no build system, package manager, bundler, test suite, linter, or CI configuration. Do not invent one unless the user asks.

## Running and verifying

- Open `index.html` directly in a browser, or serve the directory with any static server (e.g. `python3 -m http.server`).
- There are no automated tests. Verify gameplay or UI changes by loading the page in a real browser and playing through the affected path (title → start → gameplay → pause/bomb/game over as relevant).
- Check both desktop (keyboard) and touch layouts when changing controls or HUD markup, and a phone-width viewport when changing CSS.

## Architecture of `index.html`

The script is one `'use strict'` IIFE. Everything lives in closures; there are no modules and no globals to attach to. Major sections, in order:

1. **DOM references**: `const x = document.getElementById(...)` block near the top of the IIFE. Element IDs used by the script must match the markup.
2. **Persistence keys**: `STORAGE_BEST`, `STORAGE_MUTE`, `STORAGE_DIFF`, `STORAGE_REDUCED` (all under the `skyfighter.*` namespace). Reads/writes go through `localStorage`.
3. **Tuning data** (plain objects/arrays, the first place to look when changing balance):
   - `DIFFICULTIES`: `easy` / `normal` / `hard`, controlling spawn rate, enemy speed, enemy fire chance, starting lives and bombs, and boss HP multiplier.
   - `KILLS_PER_STAGE` (18): kills required to advance a stage.
   - `THEMES`: sky palettes cycled per stage.
   - `ENEMY_TYPES`: `grunt`, `interceptor` (weaving), `tank` (4 HP). `minStage` gates when each appears.
   - `BOSSES`: six bosses (VANGUARD → TITAN), each with colors and `hpMul`.
   - `BOSS_PATTERNS`: bullet-pattern functions (radial, aimed spread, columns, spiral, cross, homing, double ring) used by `updateBoss`.
4. **Audio**: `ensureAudio()` lazily creates the `AudioContext` (browsers require a user gesture); `beep()`, `noiseBurst()`, and the `sfx` object synthesize all sounds. There are no audio files.
5. **HUD and feedback helpers**: `renderHearts`, `renderBombs`, `flashHit`, `showComboPop`, `showStageBanner`, `screenShake`, `announce` (screen-reader live region).
6. **Input**: keyboard handlers on `window`, touch joystick (`moveZone`) and buttons (`fireBtn`, `bombBtn`), and mouse/pointer on the canvas.
7. **Game state and logic**: `resetGame()`, `update(dt)` (the main simulation step), `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`, `applyPowerUp`, `fireWeapon`, `useBomb`, `breakChain`/`addScore` (kill-chain multiplier), `loseLife`, `killBoss`, `endGame`, `togglePause`.
8. **Rendering**: `draw()` plus `drawPlane`, `drawBossShip`, `drawClouds`. Drawing reads state; it should not mutate it.
9. **Main loop**: `loop(now)` drives `update` and `draw` via `requestAnimationFrame`.

### Controls (keep docs and code in sync)

- Move: Arrow keys or WASD; touch joystick on the left.
- Fire: Space (held) or the fire button.
- Bomb: `X` or Shift (keyboard); bomb button on touch.
- Pause: `P` or Escape.
- Mouse/pointer also moves the ship over the canvas.

### Design constraints

- The canvas uses a fixed logical size `BASE_W × BASE_H` (480 × 720), scaled by `resize()`. Game coordinates should stay in logical units, not CSS pixels.
- Time-based updates use `dt`. Keep new movement and timers frame-rate independent.
- Respect `reducedMotion`: screen shake and flashes should be gated on it.
- Keep the game playable with no audio (muted state) and on touch devices.

## Conventions

- Keep everything in `index.html` unless the user asks for a split. If the file is split later, update this section.
- Match the existing style: 2-space indentation, `const` by default, arrow functions for callbacks, descriptive camelCase names, and `UPPER_SNAKE` for constant tables.
- Balance changes belong in the tuning data objects above, not scattered through logic.
- Don't add dependencies, frameworks, or external CDN scripts without an explicit request. The game is intentionally dependency-free.
- Keep user-facing text short; the HUD is tight on phone-sized screens.

## Git

- Development branch for this work: `claude/gifted-tesla-16rh3c`. Commit and push there; do not push to other branches without permission.
- Do not open pull requests unless explicitly asked.
- Commit messages should describe the gameplay or UI change in the imperative mood, with a body for non-trivial changes.

## Things to avoid

- Don't commit `.env`, credentials, or local settings files.
- Don't rewrite or reformat the whole `index.html` for a small change; keep diffs focused so they're reviewable.
- Don't replace `localStorage` keys without a migration plan, since they hold players' saved best score and settings.
