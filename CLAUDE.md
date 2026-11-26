# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Google_4 is a single-page browser game, **Sky Fighter**: a Raiden-style vertical shoot-'em-up rendered on an HTML `<canvas>`. The whole game lives in one file:

- `index.html` (~1,670 lines) — inline `<style>`, markup, and one inline `<script>` IIFE containing all game logic.
- `README.md` — a stub with no real project documentation.

There is no build step, no package manager, no bundler, no test suite, and no linter configuration. Nothing is installed or compiled; the file is served as-is.

## Running and testing

- **Run:** open `index.html` directly in a browser, or serve the directory with any static server (e.g. `python3 -m http.server`) and visit it.
- **Tests:** none exist. Verify changes by playing the game in a browser. For automated checks, use the pre-installed Playwright/Chromium (see the environment notes) rather than adding a test framework unless asked.
- **Syntax sanity check:** after editing the script, you can extract the `<script>` body and run `node --check` on it to catch parse errors quickly.

## Code layout inside `index.html`

The file is organized top-down:

1. **`<style>`** — dark-scheme styles. HUD (`#hud`, `.stat`, hearts/bombs), boss bar (`#bossBarWrap`), touch controls (`#touchControls`), overlay/title card, vignette and flash effects, and a `.sr-only` helper for screen readers.
2. **Markup** — `#gameCanvas`, HUD, boss bar, combo/stage banners, top bar buttons (`#muteBtn`, `#motionBtn`, `#pauseBtn`), touch controls, and the `#overlay` start/pause/game-over card with difficulty buttons (`.diffBtn`).
3. **Script constants and config:**
   - `STORAGE_*` keys (`localStorage`, prefixed `skyfighter.`): best score, mute, difficulty, reduced motion.
   - `DIFFICULTIES` — `easy` / `normal` / `hard` tuning: spawn rates, enemy speed, shoot chance, lives, bombs, boss HP multiplier.
   - `KILLS_PER_STAGE` — kills required before a stage boss spawns.
   - `THEMES` — six stage sky palettes; the theme cycles with the stage (`(stage - 1) % THEMES.length`).
   - `ENEMY_TYPES` — `grunt`, `interceptor` (weaving), `tank` (4 HP, appears from stage 2).
   - `BOSSES` — six boss designs (name, colors, HP multiplier).
   - `BOSS_PATTERNS` — attack pattern functions (`fireRadial`, `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`, `fireCross`, `fireHomingSeeded`, `fireDoubleRing`).
4. **Audio** — a Web Audio `AudioContext` created lazily by `ensureAudio()` on first user gesture. Sound effects are a table `sfx` built from `beep()` and `noiseBurst()`.
5. **Input** — keyboard (`keys[e.code]`), mouse steering and click-to-fire on the canvas, and touch joystick (`moveZone`), `FIRE`, and `BOMB` buttons.
6. **Game state** — module-level `let` variables (`player`, `bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`, `boss`, `score`, `lives`, `bombCount`, `stage`, `chain`, `multiplier`, etc.). `resetGame()` reinitializes them for a new run.
7. **Update and draw** — `update(dt)` advances the simulation (movement, spawning, collisions, power-ups, boss logic). `draw()` renders each frame. `loop(now)` drives both via `requestAnimationFrame`, clamps `dt` to 0.05 s, and skips updates while paused or not running. `trackPerf()` adjusts a `quality` level (0 full, 1 medium, 2 low) based on frame timing.
8. **Scoring** — `addScore()` applies the kill-chain multiplier; `breakChain()` resets it when the chain times out.

## Key game mechanics

- **Stages:** clearing `KILLS_PER_STAGE` triggers a boss; defeating it advances the stage and the sky theme.
- **Weapons:** `vulcan` and `laser`, each with levels 1–3. Weapon power-ups raise the level; past level 3 they swap weapon type and reset to level 1 (`applyPowerUp`).
- **Power-ups:** `life` (up to `maxLives`), `bomb` (up to `maxBombs`), and `weapon`.
- **Bombs:** `useBomb()` clears all enemy bullets, destroys every regular enemy on screen, and deals 24 damage to any active boss.
- **Pause:** `P` or `Esc` on desktop, pause button on the top bar; the start button also resumes a paused game.

## Conventions

- **Single file, no dependencies.** Keep everything inline in `index.html`. Do not add npm packages, external CDN scripts, or a build system unless the user asks.
- **Vanilla JS in an IIFE with `'use strict'`.** Follow the existing style: `const` for config, `let` for mutable state, `camelCase` identifiers, `UPPER_SNAKE_CASE` for config tables and storage keys.
- **Preserve the accessibility and motion settings.** Screen shake, flash, and pulse effects go through `screenShake()`, `flashHit()`, and `flashBomb()`, which respect `reducedMotion`. New visual effects should do the same. Announcements for screen readers go through `announce()` into `#srLive`.
- **Keep both input paths working.** Any new control must work with keyboard, mouse, and touch (`#touchControls`). The mobile layout uses `touch-action: none` and must stay usable at phone width.
- **Persist settings through the existing keys.** Use the `STORAGE_*` constants and wrap storage access so the game still runs when `localStorage` is unavailable.
- **Use `dt`-based movement.** Speeds are in pixels per second scaled by `dt`, so gameplay stays consistent across frame rates.
- **Canvas logical size is fixed at `BASE_W` × `BASE_H` (480 × 720).** `resize()` sets the CSS display size to fit the window (capped at 1.6× scale). Game logic always works in base coordinates.

## When making changes

- Read the relevant section of `index.html` before editing; line numbers shift, so search by function or constant name.
- Keep changes small and scoped. Don't reformat unrelated code.
- Check the game in a browser after gameplay changes: start, play through a kill threshold to a boss, use a bomb, pause/resume, and confirm the HUD and game-over card update.
- Commit to the working branch and push; do not open a pull request unless asked.
