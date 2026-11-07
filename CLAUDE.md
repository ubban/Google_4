# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Google_4 is a small static web project. Its only real content is **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs in the browser on a single HTML5 `<canvas>`.

Files:

- `index.html` (~1,670 lines): the entire game. Inline CSS in `<style>` (lines ~10–359), markup for the HUD/overlays/touch controls, and one inline `<script>` (starts ~line 422) that wraps everything in an IIFE.
- `README.md`: a placeholder title; not documentation of the game.

There is no build step, no package manager, no bundler, no tests, and no linter configured. Nothing needs to be installed.

## Running and testing

- Run: open `index.html` directly in a browser, or serve the directory with any static server (for example `python3 -m http.server` and visit `http://localhost:8000`).
- Test: there is no automated test suite. Verify changes by playing the game in a browser: start a game on each difficulty, check weapon and power-up pickup, bombs, boss fights, pause/resume, game over, and the best-score persistence. Also check the layout at phone width, since the game supports touch controls.
- Syntax check after editing JS: extract the `<script>` body and run `node --check` on it, or load the page and watch the browser console for errors.

## Architecture of `index.html`

Everything runs inside one `(function () { 'use strict'; ... })()` block. Key parts, in file order:

- **DOM references**: `const` lookups at the top of the script (HUD elements, overlay, touch controls, buttons).
- **Persistence**: `localStorage` keys are namespaced under `skyfighter.`: `bestScore`, `muted`, `difficulty`, `reducedMotion`. Every read/write should tolerate missing or blocked storage the same way the existing code does.
- **Adaptive quality**: `trackPerf(dt)` samples the last 45 frame times and steps `quality` between 0 (full), 1 (medium), and 2 (low) when average frame time drifts out of a target band.
- **Difficulty table** (`DIFFICULTIES`): `easy` / `normal` / `hard` control spawn timing, enemy speed, enemy shot chance, starting lives, starting bombs, and a boss HP multiplier.
- **Tuning tables**: `KILLS_PER_STAGE` (18), `THEMES` (six sky palettes that cycle with stage), `ENEMY_TYPES` (`grunt`, `interceptor`, `tank`, with `minStage` gating), and `BOSSES` (six bosses, each with colors and an `hpMul`).
- **Audio**: lazily created with `ensureAudio()` (called from the first user gesture). Sound effects are synthesized with `beep()` and `noiseBurst()`; there are no audio files. `muted` silences them.
- **Layout and input**: `BASE_W = 480`, `BASE_H = 720` define the logical play field, and `resize()` scales the canvas to fit. Keyboard state is kept in `keys[e.code]`. Mouse steers and clicks to fire on the canvas. Touch uses a left-side joystick (`moveZone`) plus FIRE and BOMB buttons.
- **Game state**: `resetGame()` initializes everything. `update(dt)` advances the simulation, `draw()` renders, and `loop(now)` drives both through `requestAnimationFrame`. `dt` is clamped to 50 ms so tab switches don't teleport objects.
- **Weapons**: two families, `vulcan` (spread shots) and `laser` (piercing lanes), each with levels 1–3. `fireWeapon()` implements them. Weapon power-ups (`applyPowerUp`) raise the level first; past level 3 they switch family and reset to level 1.
- **Power-ups**: `maybeSpawnPowerUp()` drops `life` (6%), `bomb` (8%), or `weapon` (20%) items.
- **Bosses**: `BOSS_PATTERNS` is an array of per-boss attack pattern functions built from helpers such as `fireRadial`, `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`, `fireCross`, `fireHomingSeeded`, and `fireDoubleRing`. `updateBoss()` runs them; `killBoss()` advances the stage.
- **Lifecycle**: `loseLife()` handles damage, `endGame()` shows the game-over card and saves a new best, and `togglePause()` is bound to P / Esc and the pause button.
- **Accessibility**: `announce()` writes to the `#srLive` ARIA live region for key events. Reduced motion (`reducedMotion`) disables screen shake, flashes, and the vignette pulse; it defaults to the OS `prefers-reduced-motion` setting until the user toggles it.

## Conventions

- Keep the game in a single self-contained `index.html`. Don't split it into modules or add a build step unless the user asks; the project is deliberately zero-dependency.
- No external libraries or CDN links. Everything, including sound and graphics, is generated in code.
- Keep the code inside the existing IIFE. Don't add new globals.
- Match the existing style: 2-space indentation, `const`/`let`, arrow functions for callbacks, and short helper functions named in camelCase.
- Put new tuning values in the existing tables (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `THEMES`) rather than hard-coding them in logic.
- Any new `localStorage` access should be wrapped the same way as the existing reads and writes.
- Preserve mobile behavior: keep the touch controls working and respect `touch-action: none` on the play area.

## Git

- Development branch for this work: `claude/gifted-tesla-f0t0l5`. Commit and push there; don't push to other branches without explicit permission.
- Push with `git push -u origin <branch>`.
- Write commit messages in the imperative mood with a short summary line. Describe gameplay changes in terms of player-visible effect.
