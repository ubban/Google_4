# CLAUDE.md

Guidance for working in this repository.

## Project overview

Google_4 is a small static web project. Its only game is **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs in the browser on an HTML5 `<canvas>`.

- `index.html` — the entire game: markup, CSS, and one inline `<script>` IIFE. No build step, no bundler, no dependencies, no network calls.
- `README.md` — a one-line title. Keep it minimal unless asked to expand it.

There is no `package.json`, no test suite, and no linter configured.

## Running it

Open `index.html` directly in a browser, or serve the directory with any static server (for example `python3 -m http.server`). Nothing needs to be installed.

## Verifying changes

There is no automated test or lint command. Verify changes by:

1. Loading `index.html` in a browser and checking the browser console for errors.
2. Exercising the affected feature by hand: start a game, fire, use a bomb, collect power-ups, clear a stage, reach a boss, and die. Check the game-over screen and best score.
3. Checking touch controls and layout at phone width when HUD or overlay CSS changes. The viewport is locked (`user-scalable=no`, `overflow: hidden`), so confirm nothing gets clipped.

For syntax-only checks on the inline script, extract it and run `node --check` on the extracted file.

## Code structure (`index.html`)

The file is organized in order: `<style>` block, overlay/HUD/touch markup, then one `<script>` wrapped in `(function () { 'use strict'; ... })()`. Inside the script, sections are marked with comment banners like `// ---------- Input ----------`. Keep that style when adding sections.

Key areas, roughly in file order:

- **DOM references and storage keys** — `STORAGE_*` constants prefixed `skyfighter.`.
- **Adaptive performance** — `trackPerf` moves a `quality` level (0–2) up or down based on average frame time over 45 frames.
- **Tuning tables** — `DIFFICULTIES`, `THEMES` (stage backdrops, cycled with `(stage - 1) % THEMES.length`), `ENEMY_TYPES`, `BOSSES`, `KILLS_PER_STAGE`. Balance changes usually belong here, not in the update logic.
- **Audio** — `ensureAudio()` creates the `AudioContext` lazily. It must run from a user gesture. Sound effects go through `beep` / `noiseBurst`; `muted` is persisted.
- **Input** — keyboard state is tracked by `e.code` in `keys`. Touch uses a joystick on the left (`moveZone`) plus `fireBtn` and `bombBtn`. Mouse steers via `canvasPointFromEvent`.
- **Game state** — `resetGame()` initializes every per-run variable. Any new run-scoped state must be reset there.
- **Core loop** — `update(dt)` advances the simulation, `draw()` renders, and `loop(now)` drives both via `requestAnimationFrame`. Pausing is handled by `togglePause()`.
- **Weapons and power-ups** — `fireWeapon()` implements Vulcan and Laser, each with 3 levels. `applyPowerUp()` handles `life`, `bomb`, and `weapon` pickups. The weapon power-up upgrades level first, then toggles between Vulcan and Laser and resets to level 1.
- **Bosses** — `spawnBoss()`, `updateBoss()`, and `killBoss()`. `BOSS_PATTERNS` holds the bullet patterns. `killBoss()` advances `stage`.
- **Run end** — `loseLife()` and `endGame()`. `endGame()` writes the best score to `localStorage`.

## Conventions and gotchas

- **Single file, no dependencies.** Don't add libraries, a build step, or extra files unless the user asks. Keep logic inside the existing IIFE; don't add globals.
- **Fixed logical resolution.** The game simulates at `BASE_W × BASE_H` (480×720). `resize()` scales only the CSS size. Coordinates and sizes in game code are in logical pixels, so don't size things from `window` dimensions.
- **Timers guarded by state.** Some boss-spawn and stage-clear logic uses `setTimeout`. Check `running` and `gameOver` inside the callback, as the existing code does, so a callback can't fire after a run ends.
- **Reduced motion.** `reducedMotion` is seeded from `prefers-reduced-motion` unless the user has set a preference. Any new screen shake, flash, or heavy animation should respect it.
- **Accessibility announcements.** Use `announce(text)` to update the `aria-live` region for important events (stage change, life lost, game over). Keep the text short.
- **Persisted settings.** Best score, mute, difficulty, and reduced motion live in `localStorage`. Wrap any new storage use so the game still works when storage is unavailable, since the existing reads don't guard against that.
- **Keep the existing style.** 2-space indentation, single quotes in JS, `const` by default, short comments that explain why rather than what. Match the surrounding code's density and naming.

## Git workflow

- Commit with clear, descriptive messages that say what changed and why.
- Develop on the branch assigned for the session. Don't push to `main` without explicit permission.
- Don't open a pull request unless asked.
