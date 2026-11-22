# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo is a single-file browser game, **Sky Fighter**: a Raiden-style vertical shoot-'em-up built on HTML5 Canvas.

- `index.html` is the entire game: markup, CSS (inside `<style>`), and JavaScript (one IIFE inside `<script>`). No frameworks, no bundler, no npm.
- `README.md` is a stub and contains no setup instructions.
- There is no build step, no test suite, and no linter configured.

## Running the game

Open `index.html` directly in a browser, or serve the directory with any static server (for example `python3 -m http.server`). No install is needed.

## Code layout inside `index.html`

The script is organized by section comments. Approximate landmarks (line numbers drift as the file changes):

- **DOM refs and constants** (~line 423–520): `canvas`, HUD and overlay element lookups; `STORAGE_*` keys; `DIFFICULTIES`; `KILLS_PER_STAGE`; `THEMES` (per-stage sky palettes); `ENEMY_TYPES`; `BOSSES`.
- **Persistence**: best score, mute, difficulty, and reduced-motion are stored in `localStorage` under `skyfighter.*` keys. Always guard reads and writes with the existing `Number(...) || 0` style fallbacks so a missing or corrupt value does not break startup.
- **Audio**: `ensureAudio()`, `beep()`, `noiseBurst()`, and a `sfx` object. Audio is created lazily on first user gesture because browsers block autoplay.
- **Input**: keyboard (`keys` map keyed by `e.code`), mouse on the canvas, and a touch joystick plus FIRE and BOMB buttons. The touch controls are shown only on touch devices.
- **Game state and loop**: `resetGame()`, `update(dt)`, `draw()`, and `loop(now)` driven by `requestAnimationFrame`. `update` takes a delta time, so keep per-frame motion dt-scaled.
- **Weapons and power-ups**: `fireWeapon()` (Vulcan and Laser, each with 3 levels), `applyPowerUp()` (`life`, `bomb`, `weapon`), and `useBomb()`.
- **Bosses**: `spawnBoss()`, `updateBoss()`, `killBoss()`, and `BOSS_PATTERNS` (radial, aimed spread, columns, spiral, cross, homing, double ring).
- **Scoring**: `addScore()` applies a kill-chain multiplier. `breakChain()` resets it.
- **Stages**: each stage is `KILLS_PER_STAGE` kills, ending in a boss. Themes cycle with the stage.

## Gameplay facts to keep consistent

- Difficulty (`easy` / `normal` / `hard`) sets spawn rate, enemy speed, enemy fire chance, starting lives, starting bombs, and boss HP multiplier.
- Controls: arrows or WASD to move, Space to fire (hold to keep firing), X or Shift for bomb, P or Esc to pause. Mouse steers and clicks to fire. Touch uses a left-side drag joystick plus FIRE and BOMB buttons.
- The control hint text in the `#controlsHint` element must match the actual bindings. Update it whenever you change input handling.

## Conventions

- Keep the game in one file unless the user asks for a split. Do not add a build system or dependencies without a clear reason.
- Match the existing style: 2-space indent, `const`/`let`, arrow functions for callbacks, section comments with `// ---------- Name ----------`.
- Do not introduce external network requests or CDN assets. The game is self-contained.
- Keep the mobile layout working. The canvas scales to fit, the HUD wraps, and `touch-action: none` is set on the body. Check phone-width behavior after changing layout or controls.
- Respect the reduced-motion setting when adding screen shake, flashes, or pulsing effects. `reducedMotion` gates these.
- Any new user-facing text should also be announced through the `srLive` live region when it conveys game state, so screen readers get it.

## Verifying changes

There are no automated tests. To verify a change:

1. Open `index.html` in a browser (or serve it locally) and check the browser console for errors.
2. Play through at least one full stage to a boss on `normal`, and confirm the HUD, weapon level, bombs, and score update.
3. Check pause (P / Esc), mute, difficulty switching, and the game-over overlay and best-score update.
4. For layout or input changes, also check a narrow (phone-width) viewport and touch controls.

If you add a test harness or tooling later, document the commands here.

## Git

- Development branch for this work: `claude/gifted-tesla-uc0wt4`. Commit and push there unless told otherwise.
- Keep commit messages short and descriptive, in the imperative mood, matching the existing history (for example, `Add Sky Fighter: ...`).
