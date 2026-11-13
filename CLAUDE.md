# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo contains **Sky Fighter**, a single-page HTML5 canvas vertical shoot-'em-up in the style of Raiden. The entire game lives in one file, `index.html` (~1,670 lines): inline `<style>`, markup, and one inline `<script>` wrapped in an IIFE. `README.md` is a placeholder.

There is no build step, package manager, bundler, framework, or test suite. Nothing needs to be installed.

## Running and verifying

- Run: open `index.html` directly in a browser, or serve the directory with any static server (for example `python3 -m http.server`).
- There are no automated tests or linters configured. Verify changes by loading the page and playing: start a game, fire, use a bomb, pause, change difficulty, and check the mobile layout at phone width.
- For a quick syntax check of the script, extract the contents of the `<script>` block and run `node --check` on it.

## File layout (`index.html`)

1. **`<head>`**: meta tags (viewport disables user zoom, `theme-color` `#0b1d3a`), `<title>`, and the full stylesheet.
2. **Markup**: `#container` holds `#gameCanvas`, the HUD (`#hud`: score, best, multiplier, weapon, hearts, bombs), the boss bar, combo and stage banners, `#topBar` (mute, reduced-motion, pause), touch controls (`#touchControls`), and `#overlay` (title card, difficulty picker, Start button, control hints).
3. **Script** (single IIFE, all state is module-private):
   - **Persistence keys** (`localStorage`, prefixed `skyfighter.`): `bestScore`, `muted`, `difficulty`, `reducedMotion`. Reads are guarded so the game still works if storage is unavailable.
   - **Config tables**: `DIFFICULTIES` (spawn rate, enemy speed, shoot chance, lives, bombs, boss HP multiplier), `THEMES` (six stage backdrops, cycled per stage), `ENEMY_TYPES` (grunt, interceptor, tank), `BOSSES` (six bosses), `KILLS_PER_STAGE` (18).
   - **Audio**: a lazily created Web Audio context (`ensureAudio`) with `beep` and `noiseBurst` helpers. Sound is generated, not loaded from files.
   - **Input**: keyboard (`keys` map, keyed by `e.code`), mouse steering and click-to-fire (`canvasPointFromEvent`), and a touch joystick plus FIRE and BOMB buttons.
   - **Game state**: module-level `let` variables reset by `resetGame()`. The main loop is `update(dt)` followed by `draw()`, driven by `requestAnimationFrame` in `loop(now)`. `dt` is clamped to 0.05s.
   - **Systems**: enemy spawning (`spawnEnemy`, `spawnBoss`), power-ups (`maybeSpawnPowerUp`, `applyPowerUp`), weapons (`fireWeapon`; vulcan and laser, each with levels 1 to 3), bomb (`useBomb`), kill-chain score multiplier (`addScore`, `breakChain`), boss patterns (`BOSS_PATTERNS`, `updateBoss`, `killBoss`), lives (`loseLife`), and `endGame`.
   - **Rendering**: canvas drawing for planes, bosses, clouds, and explosions. `quality` (0 full, 1 medium, 2 low) is adjusted by `trackPerf` to keep frame time acceptable.

## Conventions

- Keep the game as one self-contained HTML file unless the user asks for a restructure. Do not add a build step or external dependencies without a clear reason.
- Game tuning lives in the config tables at the top of the script. Prefer editing those over hard-coding numbers in systems.
- New HUD or UI text should go through the existing `announce()` helper or the `#srLive` live region where it matters for screen readers.
- Respect the reduced-motion setting (`reducedMotion`) for any new screen shake, flashes, or pulsing effects.
- Controls must keep working on touch devices as well as keyboard and mouse. Layout must work at phone width with no horizontal scroll.
- Keep all game logic and state inside the existing IIFE; do not add globals.

## Git

- Development branch for this work: `claude/gifted-tesla-rmbezb`. Push there with `git push -u origin <branch>`. Do not push to other branches without permission.
- Commit messages should describe the gameplay or code change, in the imperative mood, with a short body where useful.
- Do not open a pull request unless explicitly asked.
