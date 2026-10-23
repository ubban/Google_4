# CLAUDE.md

Guidance for Claude Code when working in this repository.

## What this repo is

A single-page HTML5 canvas game, **Sky Fighter**: a Raiden-style vertical shoot-'em-up.
The whole game lives in `index.html` (~1670 lines): markup, CSS in a `<style>` block, and
one inline `<script>` that wraps everything in an IIFE with `'use strict'`.

- `index.html` - the entire game
- `README.md` - placeholder; contains no project documentation yet

There is no build step, no package manager, no bundler, no test suite, and no external
dependencies. Do not introduce any of these unless the user asks.

## Running and checking changes

- Open `index.html` directly in a browser. There is no dev server.
- No automated tests exist. Verify behavior by playing the game in a browser after a change,
  paying particular attention to input (keyboard, mouse, and touch), HUD updates, and the
  game-over and best-score flow.
- Quick syntax check of the inline script without a browser (needs Node):
  `node -e "new Function(require('fs').readFileSync('index.html','utf8').split('<script>')[1].split('</script>')[0]); console.log('syntax ok')"`

## Layout of `index.html`

1. **`<style>`** - HUD pill styling, glassmorphic overlay card, touch controls, and
   `body.reduced-motion` rules.
2. **Markup** - `<canvas id="gameCanvas">`, the overlay (title, stats, difficulty buttons,
   start button), HUD elements (score, best, multiplier, weapon, hearts, bombs, boss bar),
   touch controls (`#touchControls`, `#moveZone`, `#fireBtn`, `#bombBtn`), and an
   accessibility live region (`#srLive`).
3. **Script** - sections are marked with `// ---------- Name ----------` banners, in order:
   - DOM lookups and constants (`STORAGE_*` keys)
   - `announce()` and adaptive performance (`trackPerf` drops render quality when frames are slow)
   - Config tables: `DIFFICULTIES`, `THEMES`, `ENEMY_TYPES`, `BOSSES`, `KILLS_PER_STAGE`
   - Persisted settings: best score, mute, difficulty, reduced motion
   - Audio (`ensureAudio`, `beep`, `noiseBurst`, `sfx`), HUD helpers, screen shake
   - `resize()` - fixed 480x720 logical canvas, scaled to fit the window
   - Input - keyboard, touch joystick, fire and bomb buttons, mouse steering
   - Game state (`resetGame`), drawing helpers (`drawPlane`, `drawBossShip`), spawning
   - `update(dt)` - the main simulation step
   - Power-ups (`applyPowerUp`), weapons (`fireWeapon`), boss attack patterns
     (`BOSS_PATTERNS`, `updateBoss`, `killBoss`), life loss, `endGame`, pause
   - `draw()` and `loop(now)` driven by `requestAnimationFrame`

## Game rules to keep in mind

- **Difficulty** (`DIFFICULTIES`): `easy`, `normal` (default), `hard`. Each sets spawn
  rates, enemy speed, enemy shoot chance, starting lives, starting bombs, and boss HP multiplier.
- **Stages**: each stage ends after `KILLS_PER_STAGE` (18) kills, which triggers a boss
  roughly 1.2 s later. `THEMES` cycles the background palette per stage.
- **Enemies**: `grunt` (stage 1+), `interceptor` (weaving, stage 1+), `tank` (4 HP, stage 2+).
- **Power-up drops** (`maybeSpawnPowerUp`): one roll per kill. Life 6%, bomb 8%, weapon 20%,
  otherwise nothing.
- **Weapons**: `vulcan` and `laser`, each with levels 1-3. Collecting a weapon pickup at
  max level swaps the weapon and resets the level to 1.
- **Bombs**: clear the screen and damage bosses. Capped by difficulty's `bombs` value.
- **Score**: kill-chain multiplier (`chain`, `chainTimer`, `multiplier`) builds on
  consecutive kills and breaks on a timeout or a hit.
- **Best score** is saved to `localStorage` only on game over when it is beaten.

## Persistence keys

All `localStorage` keys are prefixed `skyfighter.`:
`skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`,
`skyfighter.reducedMotion`. Reads and writes are not wrapped in try/catch today; if you
add new storage use, wrap it so the game still runs when storage is blocked.

## Controls

- Keyboard: arrows or WASD to move; Space to fire (held); X, Shift to bomb; P or Escape to pause.
- Mouse: hover the canvas to glide the plane toward the pointer; mouse down fires.
- Touch: left-side joystick to move, fire button, bomb button. Shown when the device supports touch.
- Keyboard and joystick input take priority over mouse steering so they never fight.

## Conventions

- Match the existing style: 2-space indentation, single quotes, `const`/`let`, arrow functions
  for callbacks, short `// ----------` section banners, and terse comments that explain *why*.
- Keep everything inside the existing IIFE. Do not add globals.
- Keep game tuning in the config tables (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `THEMES`)
  rather than hard-coding magic numbers in update or draw code.
- Keep the game playable at phone width; the canvas scales through `resize()`. Preserve the
  `prefers-reduced-motion` and `reducedMotion` behavior when adding screen effects.
- Do not add external CDN scripts or fonts unless requested.

## Git

- Development branch for this work: `claude/gifted-tesla-dlg1rp`. Push there with
  `git push -u origin <branch>`.
- Do not open a pull request unless the user asks for one.
