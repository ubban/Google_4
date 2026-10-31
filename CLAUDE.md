# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a single-page HTML5 canvas game, **Sky Fighter**: a Raiden-style vertical shoot-em-up. The entire game lives in one file, `index.html` (~1,670 lines: inline CSS, markup, and a single inline `<script>`). There is no build step, no bundler, no package manager, and no external assets. All sound is synthesized with the WebAudio API.

Files:

| Path | Purpose |
| --- | --- |
| `index.html` | The whole game: styles (`<style>`, ~lines 10–359), DOM markup (~lines 360–421), and game code (`<script>`, ~lines 422–1667) |
| `README.md` | Placeholder title only |
| `CLAUDE.md` | This file |

## Running and testing

- **Run:** open `index.html` directly in a modern browser. No server is required.
- **Tests:** none exist. There is no test runner, linter, or CI configuration in the repo.
- **Verification:** because there are no automated tests, verify changes by playing the game in a browser. For gameplay or rendering changes, check at least: start, fire, bomb, a stage transition, a boss fight, game over, and the best-score update.
- For a mobile layout change, check touch controls at a phone-sized viewport as well as desktop.

## Code architecture (inside `index.html`)

The script is one IIFE-style block of `const`/`let` declarations and functions. Major sections, in order:

1. **DOM references** – `getElementById` lookups for the canvas, HUD, overlay, and touch controls.
2. **Persistence keys** – `STORAGE_BEST`, `STORAGE_MUTE`, `STORAGE_DIFF`, `STORAGE_REDUCED` (all `skyfighter.*` in `localStorage`).
3. **Quality governor** – `trackPerf(dt)` samples frame times and steps `quality` (0 full, 1 medium, 2 low) to scale particle counts.
4. **Tuning data** (plain objects, the main place to adjust balance):
   - `DIFFICULTIES` – `easy` / `normal` / `hard`: spawn rates, enemy speed, shoot chance, starting lives, starting bombs, boss HP multiplier.
   - `KILLS_PER_STAGE` – kills required before a boss spawns (currently 18).
   - `THEMES` – six sky palettes, cycled by `(stage - 1) % THEMES.length`.
   - `ENEMY_TYPES` – `grunt`, `interceptor` (weaving), `tank` (4 HP, unlocks at stage 2). Each has `minStage`.
   - `BOSSES` – six boss designs, cycled by `(stage - 1) % BOSSES.length`, each with an `hpMul`.
5. **Settings** – mute, reduced-motion (`prefers-reduced-motion` is the default if no saved preference), and difficulty, all persisted.
6. **Audio** – `ensureAudio()` lazily creates the `AudioContext` on first user gesture. `beep()`, `noiseBurst()`, and the `sfx` table build every sound from oscillators and noise.
7. **Layout** – fixed logical canvas of **480×720** (`BASE_W`, `BASE_H`), scaled to fit the window (capped at 1.6×).
8. **Input** – keyboard (`keys[e.code]`), mouse (steer toward the cursor, click to fire), and touch (left-side joystick, FIRE and BOMB buttons).
9. **Game state** – module-level `let`s: `player`, `bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`, `boss`, `score`, `lives`, `bombCount`, `stage`, `kills`, `chain`/`multiplier`, and so on. `resetGame()` reinitializes all of them.
10. **Entity logic** – `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`, `fireWeapon` (Vulcan has up to 3 spread levels; Laser fires faster), `useBomb` (clears the screen and damages bosses), `addScore` / `breakChain` (kill-chain multiplier), `loseLife`, `killBoss`, `endGame`, `togglePause`.
11. **Update / draw** – `update(dt)` advances the simulation; `draw()` renders everything with 2D canvas primitives (`drawPlane`, `drawBossShip`, `drawClouds`, and so on).
12. **Main loop** – `loop(now)` runs on `requestAnimationFrame`, calls `update` then `draw`, and clamps `dt`.

### Gameplay rules worth knowing

- Stage progression: kill `KILLS_PER_STAGE` enemies → a boss spawns 1.2 s later → defeating it grants 500 points, increments `stage`, and resets `stageKills`.
- Power-up drops (`maybeSpawnPowerUp`): ~6% life, ~8% bomb, ~20% weapon upgrade. A weapon pickup raises the weapon level up to 3; at level 3 it instead switches between Vulcan and Laser and resets the level to 1.
- Best score is written to `localStorage` only in `endGame()`, and only when the score beats the stored value.
- Lives and starting bombs come from the selected difficulty.

## Conventions

- **Keep it a single file.** Do not split `index.html` into modules or add a build step unless the user asks. Everything the game needs should stay inline.
- **No external dependencies or network assets.** Keep audio synthesized and graphics drawn with canvas primitives. Do not add CDN links, fonts, or image files without an explicit request.
- **Balance changes go in the tuning objects** (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `THEMES`), not scattered as magic numbers in logic.
- **Respect the reduced-motion and quality systems.** New screen shake, flashes, or particle bursts should go through `screenShake`, `flashHit`/`flashBomb`, and `spawnExplosion` so they honor `reducedMotion` and `quality`.
- **Accessibility:** announce important state changes through `announce()` (the `#srLive` live region), and keep button `title`s and labels accurate.
- **Match the existing style:** 2-space indentation, single quotes, `const` by default, and section banner comments of the form `// ---------- Name ----------`.
- Guard `localStorage` access if you add new keys. Reads currently assume it is available.

## Git workflow

- Develop on the branch assigned for the task (currently `claude/gifted-tesla-qa3f26`). Do not push to other branches without permission.
- Write clear commit messages that describe the change and its reason.
- Do not open a pull request unless the user explicitly asks for one.
