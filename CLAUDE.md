# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a single-page HTML5 canvas game, **Sky Fighter**: a Raiden-style vertical shoot-em-up. The entire game lives in `index.html` (about 1,670 lines): one `<style>` block, the markup, and one inline `<script>` wrapped in an IIFE with `'use strict'`.

`README.md` is a placeholder and does not describe the game. Treat `index.html` as the source of truth.

## Repository layout

- `index.html` - the whole game: CSS, DOM/HUD markup, and JavaScript.
- `README.md` - minimal placeholder.
- `CLAUDE.md` - this file.

There is no build step, package manager, bundler, test suite, or linter. Do not invent npm scripts or test commands. To run the game, open `index.html` in a browser, or serve the directory with any static server.

## Architecture of `index.html`

The script is organized into sections, roughly in this order:

1. **DOM references** - all `getElementById` / `querySelectorAll` lookups are grabbed once at the top.
2. **Storage keys** - `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion` in `localStorage`.
3. **Adaptive performance** - `trackPerf` samples frame times and steps a `quality` level (0 full, 1 medium, 2 low) up or down.
4. **Config tables** - the main tuning surface:
   - `DIFFICULTIES` (`easy` / `normal` / `hard`): spawn timing, enemy speed, enemy fire chance, starting lives and bombs, boss HP multiplier.
   - `KILLS_PER_STAGE` - kills needed to advance a stage.
   - `THEMES` - sky gradients, cloud and sun colors, and names, cycled per stage.
   - `ENEMY_TYPES` - `grunt`, `interceptor` (weaves), `tank` (4 HP, unlocks at stage 2).
   - `BOSSES` - six bosses with colors and `hpMul`.
5. **Audio** - a Web Audio synth (`ensureAudio`, `beep`, `noiseBurst`, and the `sfx` table). Audio is created lazily on the first user gesture.
6. **Input** - keyboard (`keys` map keyed by `e.code`), a touch joystick, fire and bomb buttons, and pointer handling for the canvas.
7. **Game state and logic** - `resetGame`, `update(dt)`, `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`, `fireWeapon`, `useBomb`, `applyPowerUp`, `loseLife`, `endGame`, `togglePause`.
8. **Boss patterns** - `BOSS_PATTERNS` is an array of bullet-pattern functions (radial, aimed spread, columns, spiral, cross, homing, double ring) that `updateBoss` cycles through.
9. **Rendering** - `draw()` renders each frame: clouds (parallax), planes, bosses, bullets, power-ups, explosions, and HUD effects. `drawPlane` and `drawBossShip` handle the ship art.
10. **Main loop** - `loop(now)` clamps `dt` to 0.05 s, runs `update` and `draw` only while running and not paused, then calls `requestAnimationFrame`.

### Game rules (for reference when changing balance)

- Weapons: `vulcan` and `laser`, each with power levels 1-3. Collecting a weapon power-up raises the level; at level 3 it toggles to the other weapon and resets to level 1.
- Power-ups: `life` (up to `maxLives`), `bomb` (up to `maxBombs`), `weapon`.
- Bombs: `X`, `Shift`, or the touch bomb button. A bomb clears the screen and damages bosses.
- Score: a kill-chain multiplier (`breakChain` resets it on damage taken).
- Pause: `P` or `Escape`.

## Conventions

- Keep the game as a single self-contained file unless the user asks to split it. Do not add a build system or external dependencies for small changes.
- Match the existing style: 2-space indentation, `const`/`let`, arrow functions for callbacks, short section comments using the `// ---------- Name ----------` banner form.
- Tune gameplay through the config tables (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `THEMES`, `KILLS_PER_STAGE`) rather than hard-coding numbers in logic.
- Respect the reduced-motion setting when adding screen effects. With `reducedMotion` on, `flashHit` and `flashBomb` return early and `screenShake` is dampened.
- Keep the touch and keyboard paths in sync when adding a control.
- Persist new user preferences under the `skyfighter.` `localStorage` prefix.
- Preserve `'use strict'` and the IIFE wrapper. Do not leak globals.

## Verification

Since there is no automated test setup, verify changes by:

1. Opening `index.html` in a browser and playing through the affected path.
2. Checking the browser console for errors.
3. Testing both keyboard and touch input when input code changes.
4. Testing at a phone-sized viewport when HUD or layout code changes.
5. Confirming that high score, mute, difficulty, and reduced-motion settings still persist across reloads.

## Git

- Commit messages should describe the gameplay or code change in plain language.
