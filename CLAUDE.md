# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo contains **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs in the browser. The whole game is a single self-contained file:

- `index.html` (~1,670 lines): markup, inline `<style>`, and one inline `<script>` IIFE. No build step, no bundler, no external dependencies.
- `README.md`: minimal project title only.

There is no package manager, test suite, linter, or CI configuration in the repo at present.

## Running and testing

- Open `index.html` directly in a browser, or serve the directory with any static server (e.g. `python3 -m http.server`).
- There are no automated tests. Verify changes by playing the game in a browser, on both desktop and a mobile-sized viewport, across all three difficulties.

## Code layout inside `index.html`

The script is one IIFE. Key sections, roughly in file order:

- **Storage keys** (`STORAGE_*`, ~line 463): `localStorage` keys for best score, mute, difficulty, and reduced motion. Keep the `skyfighter.*` prefix.
- **Difficulty table** (`DIFFICULTIES`, ~line 485): spawn rate, enemy speed, shoot chance, lives, bombs, and boss HP multiplier per difficulty.
- **Content tables**: `KILLS_PER_STAGE`, `THEMES`, `ENEMY_TYPES`, `BOSSES`, and `BOSS_PATTERNS` (bullet-pattern functions such as radial, spiral, cross, and homing).
- **Audio**: `ensureAudio()` lazily creates the `AudioContext` (browsers require a user gesture). Sound effects are synthesized with `beep()` and `noiseBurst()`; there are no audio files.
- **Input**: keyboard (`keys[e.code]`), mouse (canvas move/click), and touch (joystick zone, FIRE and BOMB buttons). Input handlers are registered near the `resize()` / `joystick*` functions.
- **Game state and logic**: `resetGame()`, `update(dt)`, `fireWeapon()`, `useBomb()`, `applyPowerUp()`, `spawnEnemy()`, `spawnBoss()`, `updateBoss()`, `killBoss()`, `loseLife()`, `endGame()`.
- **Rendering**: `draw()` and helpers `drawPlane()`, `drawBossShip()`, `drawClouds()`. HUD updates go through `renderHearts()`, `renderBombs()`, and related DOM helpers.
- **Main loop**: `loop(now)` runs under `requestAnimationFrame`. `dt` is clamped to 50 ms so pauses and tab switches don't cause big jumps. Update and draw only run when `running && !paused`.

Canvas logical size is `BASE_W = 480` x `BASE_H = 720`; `resize()` scales it to fit the viewport.

## Conventions

- Keep everything in `index.html` unless there is a clear reason to split it. Do not add a build step or external dependencies without asking.
- Use `const`/`let`, keep the existing 2-space indentation and single-quoted strings inside the script.
- Gameplay tuning numbers live in the constant tables near the top of the script. Prefer changing those over hard-coding new magic numbers in the update loop.
- User-facing preferences are persisted via `localStorage`; wrap new storage access so the game still works if storage is unavailable.
- Accessibility: respect the reduced-motion setting when adding screen shake, flashes, or particle effects. Announce important state changes through the existing `announce()` helper.
- Keep the mobile experience working: touch targets, `touch-action: none`, and safe-area insets are already set up and should be preserved.

## Controls (for reference when testing)

- Desktop: mouse to steer and click to fire; arrows/WASD to move; Space to fire; X or Shift for a bomb; P or Esc to pause.
- Mobile: drag left to fly, tap FIRE, tap BOMB.

## Git workflow

- Develop on the branch assigned to the session (currently `claude/gifted-tesla-8xqqw9`).
- Commit with clear messages describing the gameplay or code change.
- Do not open a pull request unless explicitly asked.
