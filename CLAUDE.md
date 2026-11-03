# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up that runs in the browser. The whole game is a single static file, `index.html`, with inline CSS and one inline `<script>` IIFE. There is no framework, no package manager, no build step, and no external dependencies.

Files:

- `index.html` - the entire game (markup, styles, and logic).
- `README.md` - minimal project readme.
- `CLAUDE.md` - this file.

## Running and testing

- **Run:** open `index.html` directly in a browser, or serve the directory with any static server (for example `python3 -m http.server`).
- **Build:** none. Edits take effect on reload.
- **Tests:** none exist. Verify changes by playing the game in a browser, covering the affected path (start, fire, bomb, pause, game over, and mobile touch controls when relevant).
- **Lint:** none configured.

## Code layout inside `index.html`

- **`<style>`** (roughly lines 10-360): layout, HUD, overlay card, touch controls, and the `reduced-motion` overrides. Colors and sizes are hard-coded; there are no CSS variables.
- **`<body>`** (roughly lines 361-421): the HUD (`#hud`), boss bar, combo/stage banners, top buttons (mute, reduced motion, pause), touch controls, and the start/pause overlay (`#overlay`).
- **`<script>`** (roughly lines 422-1669): one `(function () { 'use strict'; ... })();` IIFE. Its main sections, in order:
  1. DOM element references and `STORAGE_*` keys.
  2. Tuning tables: `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES`, `ENEMY_TYPES`, `BOSSES`.
  3. Audio: `ensureAudio`, `beep`, `noiseBurst`, and the `sfx` table. Audio is synthesized with the Web Audio API; there are no sound files.
  4. HUD and effects helpers: hearts, bombs, flash, combo pop, stage banner, screen shake.
  5. Input: keyboard (`keys`), virtual joystick and fire/bomb buttons for touch, and mouse.
  6. Game state and `resetGame()`.
  7. Drawing: `drawPlane`, `drawBossShip`, and `draw()`.
  8. Spawning: `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`.
  9. `update(dt)` (per-frame simulation), `updateBoss`, `endGame`.
  10. `loop(now)` runs on `requestAnimationFrame`, and `startBtn` starts a run.

## Game rules worth knowing

- **Stages:** each stage needs `KILLS_PER_STAGE` (18) kills, then a boss spawns. Boss design is `BOSSES[(stage - 1) % BOSSES.length]`. The background theme is `THEMES[...]`.
- **Enemies:** `grunt` and `interceptor` appear from stage 1; `tank` (4 HP) from stage 2.
- **Difficulty:** `easy`, `normal`, and `hard` set spawn rate, enemy speed, shot chance, starting lives, starting bombs, and boss HP multiplier.
- **Weapons:** `vulcan` and `laser`, toggled by the weapon power-up. The weapon level is shown in the HUD.
- **Power-ups:** `life`, `bomb`, and `weapon` (the `powerUps` entries use these `type` values).
- **Scoring:** kills add score with a chain multiplier that resets when the chain timer runs out.
- **Pause:** `P` or `Esc` (also the pause button).

## Persistence

Settings and the best score are stored in `localStorage` under these keys. Changing a key's name or format will reset players' saved data, so treat them as a public interface.

- `skyfighter.bestScore`
- `skyfighter.muted`
- `skyfighter.difficulty`
- `skyfighter.reducedMotion`

The reduced-motion default follows `prefers-reduced-motion` unless the player has set the value explicitly.

## Conventions

- **Keep it one file.** Do not split `index.html` into modules or add a build step unless the user asks. Everything should keep working by opening the file directly.
- **No new dependencies or CDN scripts** without an explicit request.
- **Use the existing style:** `'use strict'`, `const`/`let`, two-space indentation, single quotes in JS, and the tuning tables for any balance numbers rather than inline magic values.
- **Respect reduced motion and mute.** New visual effects should check `reducedMotion` (screen shake, flashes, and pulses are gated). New sounds should go through `sfx`/`beep` so the mute setting applies.
- **Keep touch working.** Controls must remain usable on touch devices; `touch-action` and `user-select` are disabled on purpose.
- **Accessibility:** announce important state changes through `announce()` (the `#srLive` region), and keep the `aria-live` and `title` attributes on controls.
- **Viewport:** the canvas is scaled to fit using `BASE_W` x `BASE_H` (480 x 720) and is capped at 1.6x. Keep the logical game size in those constants.

## Making changes

- Make the smallest change that does the job, and avoid unrelated refactors in the same commit.
- After editing, load the game in a browser and check the affected behavior. Watch the browser console for errors.
- Commit with a clear message that describes the gameplay or code change.
- Develop on the branch the task specifies; do not push to other branches without permission.
