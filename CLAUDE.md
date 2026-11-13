# CLAUDE.md

Guidance for Claude Code when working in this repository.

## What this repo is

`Google_4` is a small, single-file browser game: **Sky Fighter**, a Raiden-style vertical shoot-'em-up. There is no build step, no package manager, and no test suite.

## File layout

- `index.html` — the whole game: markup, CSS (inline `<style>`), and JavaScript (one inline `<script>` wrapped in an IIFE with `'use strict'`).
- `README.md` — one-line project title.
- `CLAUDE.md` — this file.

Git: the default working branch for this task is `claude/gifted-tesla-lavgyv`.

## Running it

Open `index.html` directly in a browser. No server, install, or bundler is needed.

## Code structure (inside `index.html`)

The script is organized roughly in this order:

1. **DOM lookups** — `const xEl = document.getElementById(...)` for every HUD, overlay, and touch element.
2. **Config tables** — `DIFFICULTIES` (easy/normal/hard: spawn rate, enemy speed, shoot chance, lives, bombs, boss HP multiplier), `KILLS_PER_STAGE` (18), `THEMES` (6 stage palettes, cycled by stage), `ENEMY_TYPES` (grunt, interceptor, tank), `BOSSES` (6 bosses, each with colors and an `hpMul`).
3. **Persistence** — `localStorage` keys prefixed `skyfighter.` (`bestScore`, `muted`, `difficulty`, `reducedMotion`). Every read and write is a plain `localStorage` call; keep it that way unless adding error handling for all of them.
4. **Audio** — a Web Audio context created lazily by `ensureAudio()`; sounds are built from `beep()` and `noiseBurst()`. Nothing is loaded from disk.
5. **Rendering helpers** — `renderHearts`, `renderBombs`, `flashHit`, `flashBomb`, `showComboPop`, `showStageBanner`, `screenShake`.
6. **Layout** — `resize()` scales a fixed `BASE_W = 480` × `BASE_H = 720` logical play field to the viewport.
7. **Input** — keyboard (`keys` map on `keydown`/`keyup`), touch joystick and fire/bomb buttons, and mouse (move to steer, hold to fire).
8. **Game state and loop** — `resetGame()`, `update(dt)`, `draw()`, and `loop(now)` driven by `requestAnimationFrame`. Mutable state is in `let` variables near line ~865 (`score`, `lives`, `stage`, `kills`, `running`, `paused`, `gameOver`, and so on).
9. **Weapons and power-ups** — `fireWeapon()` (vulcan / laser), `applyPowerUp()` (life, bomb, weapon swap), `maybeSpawnPowerUp()`.
10. **Bosses** — `spawnBoss()`, `updateBoss()`, `killBoss()`, and the `BOSS_PATTERNS` array of bullet-pattern functions (radial, aimed spread, columns, spiral, cross, homing, double ring).
11. **Flow** — `loseLife()`, `endGame()`, `togglePause()`.

## Controls (for reference when testing)

- Move: arrow keys or WASD; or mouse over the canvas; or touch joystick.
- Fire: Space (hold), or hold the left mouse button, or the touch fire button.
- Bomb: `X` or either Shift, or the touch bomb button.
- Pause: `P` or `Escape`.

## Conventions

- Keep the game as a single self-contained `index.html` unless the user asks for a split. Do not add a build step, framework, or external dependency without asking.
- No external network requests; the page should work offline.
- Preserve the mobile layout: `touch-action: none`, safe-area insets, and the 16px-style gutters on the HUD. Check changes at phone width.
- Respect the `reducedMotion` setting when adding new screen shake, flashes, or animations.
- Keep balance values in the config tables (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `KILLS_PER_STAGE`) rather than hard-coding them in logic.
- Use `const` by default; the existing code uses `let` only for mutable game state.
- Match the existing style: 2-space indentation, single quotes in JS, short helper functions.

## Verifying changes

There is no automated test suite. To verify a change:

1. Open `index.html` in a browser (or load it with Playwright against a `file://` URL).
2. Start a game, exercise the changed path (fire, bomb, stage transition, boss, game over).
3. Check the browser console for errors.
4. For layout changes, check both a desktop viewport and a phone-sized viewport.

## Git

- Commit with clear messages describing the gameplay or code change.
- Develop on the branch named above and push with `git push -u origin <branch>`.
- Do not open a pull request unless explicitly asked.
