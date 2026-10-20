# CLAUDE.md

Guidance for Claude Code when working in this repository.

## What this repo is

A small, dependency-free browser game, **Sky Fighter**: a Raiden-style vertical shoot-'em-up. The whole game lives in one file, `index.html` (inline `<style>` and one inline `<script>`). `README.md` is a placeholder.

There is no build step, no package manager, no bundler, no test suite, and no linter configured. Do not add any of these unless asked.

## Running it

Open `index.html` directly in a browser, or serve the directory with any static server (e.g. `python3 -m http.server`). Verify changes by playing in the browser; there are no automated checks to run.

## File layout (`index.html`)

- `<head>`: meta tags (viewport locked, `theme-color`), `<title>`.
- `<style>` (approx. lines 10–359): all CSS. Responsive layout and touch controls are styled here.
- `<body>` (approx. lines 361–421): the `#gameCanvas` element, HUD (score, best, multiplier, weapon, hearts, bombs, boss bar), `#overlay` start/pause/game-over screens, touch controls (`#touchControls`, joystick, fire and bomb buttons), and the `#srLive` aria-live region.
- `<script>` (approx. lines 422–1667): one IIFE with `'use strict'`. Nothing is exported to `window`.

Inside the script, the sections appear in this order:

1. DOM lookups and storage keys (`skyfighter.*` in `localStorage`).
2. `announce()` for screen-reader text and `trackPerf()` for adaptive quality (0 full, 1 medium, 2 low).
3. Tuning tables: `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES` (one per stage), `ENEMY_TYPES`, `BOSSES`.
4. Audio: `ensureAudio()`, `beep()`, `noiseBurst()`, and the `sfx` object. Sound is synthesized with the Web Audio API; there are no audio files.
5. HUD helpers and effects (hearts, bombs, flashes, combo pop, stage banner, screen shake).
6. Input: keyboard (`keys[e.code]`), virtual joystick, touch and mouse on the canvas.
7. Game state: `resetGame()`, `resize()` (fixed `BASE_W × BASE_H` = 480×720, scaled to fit).
8. Spawning and entities: `spawnEnemy()`, `spawnBoss()`, `maybeSpawnPowerUp()`, `spawnExplosion()`.
9. Player weapons: `fireWeapon()` (`vulcan` and `laser`, levels 1–3), `applyPowerUp()`, `useBomb()`.
10. `update(dt)`: the per-frame simulation.
11. Boss patterns: `BOSS_PATTERNS` and the `fire*` helpers, driven by `updateBoss()`.
12. Lifecycle: `loseLife()`, `killBoss()`, `endGame()`, `togglePause()`.
13. `draw()` and its helpers, then `loop(now)` (the `requestAnimationFrame` loop, `dt` clamped to 50 ms).

## Controls

- Move: arrow keys or WASD. Fire: Space (hold). Bomb: X or Shift. Pause: P or Escape.
- Touch: joystick on the left, fire and bomb buttons on the right.

## Game rules worth knowing

- Difficulty (`easy` / `normal` / `hard`) sets spawn rates, enemy speed and fire chance, starting lives and bombs, and boss HP multiplier.
- Each stage lasts until `KILLS_PER_STAGE` (18) kills, then a boss spawns 1.2 s later (`spawnBoss()`, triggered from `update()`). Stages cycle through `THEMES`.
- Tanks appear from stage 2 (`minStage`). Grunts and interceptors appear from stage 1.
- Power-up drops: `life`, `bomb`, and `weapon`. Weapon pickups raise the level to 3, then toggle between `vulcan` and `laser` and reset to level 1.
- Chain kills build a score multiplier (`breakChain()` resets it on a hit).
- The best score, mute state, difficulty, and reduced-motion preference persist in `localStorage`. Every access must tolerate storage being unavailable.

## Conventions

- Keep everything in `index.html` unless the user asks for a split. Match the existing style: 2-space indent, `const`/`let`, arrow functions for callbacks, section banner comments like `// ---------- Name ----------`.
- Tuning values live in the tables near the top of the script. Prefer changing those over hard-coding numbers in logic.
- Keep the game loop allocation-light. Entity arrays are reused; avoid creating objects or closures per frame in `update`/`draw` where an existing pattern avoids it.
- Keep the UI accessible: announce state changes through `announce()` and respect `reducedMotion` for shake and flash effects.
- Keep the layout working at phone width. Touch controls must stay usable and `preventDefault` where the existing handlers do.

## Git

- Develop on the branch the task names (currently `claude/gifted-tesla-ew8ib8`). Do not push to other branches without explicit permission.
- Commit messages are short and imperative, describing the gameplay or file change (e.g. "Add Sky Fighter: Raiden-style vertical shoot-em-up game").
