# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Sky Fighter is a Raiden-style vertical shoot-em-up that runs as a single static page on an HTML5 `<canvas>`. The whole game lives in `index.html`: CSS in a `<style>` block, markup, then one inline `<script>` wrapped in an IIFE. `README.md` is a stub.

There is no build step, package manager, bundler, linter, or test suite. To run the game, open `index.html` in a browser, or serve the directory with any static server.

## Layout of `index.html`

- **`<style>`**: HUD pills, the glassmorphic overlay card, touch controls, and the `reduced-motion` class.
- **Markup**: `#gameCanvas`, the `#overlay` start/pause/game-over card, the HUD elements, the touch controls (`#moveZone`, `#fireBtn`, `#bombBtn`), and the `#srLive` live region.
- **Script, in order**:
  1. DOM lookups and `STORAGE_*` keys.
  2. Adaptive quality (`trackPerf`), which steps `quality` between 0 and 2 based on frame time.
  3. Tuning tables: `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES`, `ENEMY_TYPES`, `BOSSES`, and `BOSS_PATTERNS` further down.
  4. Audio: `ensureAudio`, `beep`, `noiseBurst`, and the `sfx` object.
  5. Input: keyboard, touch joystick, and mouse handlers.
  6. Game state and logic: `resetGame`, `update`, `applyPowerUp`, the boss attack functions, `loseLife`, and `endGame`.
  7. Rendering: `draw` and its helpers (`drawPlane`, `drawBossShip`, `drawClouds`).
  8. `loop(now)` runs on `requestAnimationFrame`. `dt` is clamped to 50 ms.

## Game rules and where they live

- **Difficulty**: `DIFFICULTIES` sets spawn rate, enemy shoot chance, lives, starting bombs, and boss HP multiplier. The choice persists in `localStorage`.
- **Stages**: a stage ends after `KILLS_PER_STAGE` (18) kills. Each stage advances the `THEMES` palette, and every stage ends in a boss from `BOSSES`, which cycles.
- **Enemies**: `grunt`, `interceptor` (weaves), and `tank`. `tank` only appears from stage 2 (`minStage`).
- **Weapons**: `vulcan` and `laser`, each with 3 levels. A weapon power-up raises the level first, then swaps weapon and resets to level 1.
- **Power-ups**: `life`, `bomb`, and `weapon`, applied in `applyPowerUp`.
- **Score**: `addScore` applies a kill-chain multiplier, which `breakChain` resets.
- **Bombs**: `useBomb` clears the screen and damages bosses. Triggered by `X` or Shift, or by the touch bomb button.

## Controls

- Move: arrow keys or WASD. Touch uses the joystick.
- Fire: Space (held), or the mouse / touch fire button.
- Bomb: `X` or Shift.
- Pause: `P` or Escape.
- Mouse: moving over the canvas steers the ship (`mouseActive`).

## Persistence

Stored in `localStorage` under the `skyfighter.*` prefix:

- `skyfighter.bestScore`
- `skyfighter.muted`
- `skyfighter.difficulty`
- `skyfighter.reducedMotion`

Reduced motion defaults to the `prefers-reduced-motion` media query until the user toggles it. Reads of `localStorage` are not wrapped in try/catch, so a blocked-storage browser will throw at startup. Keep that in mind if you touch the top-level initialization.

## Conventions

- Keep everything in `index.html` unless there is a clear reason to split it. The single-file layout is deliberate.
- Match the existing style: 2-space indentation, single quotes in JS, `const` by default, and short section comments using the `// ---------- Name ----------` banner form.
- Tuning values are data in the tables near the top of the script. Prefer changing a table entry over adding a special case in the logic.
- Audio is generated with the Web Audio API. Do not add audio files.
- Respect the reduced-motion and mute toggles when adding effects. Screen shake and flashes should check `reducedMotion`.
- Keep the game playable with touch as well as keyboard. Any new control needs a touch path.
- Accessibility: announce important state changes through `announce()` so they reach `#srLive`.

## Verifying changes

Nothing is automated, so verify by hand:

1. Open `index.html` in a desktop browser and check the console for errors.
2. Play through at least one stage transition and one boss fight on each difficulty you touched.
3. Check the touch layout at phone width if you changed the HUD or controls.
4. Check that pause, mute, reduced-motion, and difficulty persist across a reload.

## Git

- Develop on the branch the task names. Do not push to `main` directly.
- Keep commits focused on one change, with a message that describes the gameplay or code change.
