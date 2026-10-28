# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo is **Sky Fighter**, a single-file, browser-based vertical shoot-'em-up (Raiden-style) built on the HTML5 Canvas 2D API. There is no build step, no package manager, and no test suite.

| File | Purpose |
| --- | --- |
| `index.html` | The entire game: markup (`<body>`), CSS (`<style>`), and JavaScript (`<script>`), in one ~1,670-line file |
| `README.md` | Minimal project readme (title only) |

To run it, open `index.html` in a browser. Nothing needs to be installed.

## Repository layout of `index.html`

The file is ordered top to bottom as:

1. **`<head>` / `<style>`** (lines ~1–360): CSS with colors and layout for the HUD, overlay, touch controls, and screen-reader helpers (`.sr-only`). A `body.reduced-motion` class disables visual effects.
2. **`<body>`** (lines ~361–421): the `<canvas id="gameCanvas">`, HUD elements, start/pause overlay, touch controls (`#touchControls`), and the `#srLive` ARIA live region.
3. **`<script>`** (lines ~422–1668): one IIFE (`(function () { 'use strict'; ... })();`). All state is closure-scoped; nothing is exposed on `window`.

Within the script, the main sections are, in order:

- **DOM references and constants**: `const xEl = document.getElementById(...)`, `STORAGE_*` keys, `DIFFICULTIES`, `THEMES` (stage backgrounds), `ENEMY_TYPES`, `BOSSES`.
- **Adaptive performance**: `trackPerf()` samples frame times and steps `quality` (0 full, 1 medium, 2 low) to keep frame rate up.
- **Settings persistence**: best score, mute, difficulty, and reduced motion are read from and written to `localStorage`.
- **Audio**: a lazily created `AudioContext` (`ensureAudio`), with `beep()` and `noiseBurst()` building blocks and a `sfx` object of named sound effects.
- **HUD and FX**: `renderHearts`, `renderBombs`, `flashHit`, `showComboPop`, `showStageBanner`, `screenShake`.
- **Input**: keyboard (`keys` map), touch joystick and fire/bomb buttons, and mouse drag on the canvas.
- **Game state**: `resetGame()` initializes `player`, `bullets`, `enemies`, `enemyBullets`, `powerUps`, and the scoring/chain/stage counters.
- **Drawing**: `drawPlane`, `drawBossShip`, `drawClouds`, and `draw()`, which renders one frame.
- **Spawning and combat**: `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`, `addScore`, `breakChain`, `useBomb`, `fireWeapon`, and the `fire*` bullet-pattern helpers.
- **Bosses**: `BOSS_PATTERNS` (an array of attack functions), `updateBoss`, `killBoss`.
- **Update loop**: `update(dt)` handles movement, collisions, and entity cleanup. `loop(now)` runs once per animation frame and calls `trackPerf`, `update`, and `draw`.
- **Flow control**: `loseLife`, `endGame`, `togglePause`, and the start button handler.

## Controls

| Action | Keyboard | Touch |
| --- | --- | --- |
| Move | Arrow keys / WASD | Left-side joystick (drag) |
| Fire | Hold `Space` (auto-fire while held) | Hold the FIRE button |
| Bomb | `X` or `Shift` | BOMB button |
| Pause | `P` or `Esc` | On-screen pause button (`#pauseBtn`) |

Mouse: move the pointer over the canvas to steer, and click to fire (`#controlsHint` in the markup says the same).

## Game rules and tuning

- **Difficulty** (`DIFFICULTIES`): `easy`, `normal` (default), `hard`. Each sets spawn timing, enemy speed, enemy fire chance, starting lives, starting bombs, and boss HP multiplier.
- **Stages**: every `KILLS_PER_STAGE` (18) kills triggers a boss fight about 1.2 s later (`spawnBoss`). Beating the boss advances the stage and announces the next theme. Stage `n` uses `THEMES[(n - 1) % THEMES.length]` and `BOSSES[(n - 1) % BOSSES.length]`.
- **Enemies**: `grunt`, `interceptor` (fast, weaves), and `tank` (4 HP, appears from stage 2). See `ENEMY_TYPES`.
- **Scoring**: `addScore(base)` adds `base * multiplier` to the score. Each kill increments `chain`, and the multiplier rises by 1 every 5 chained kills, up to 10x. The chain resets via `breakChain()` when its timer runs out or the player is hit.
- **Power-ups** drop from `maybeSpawnPowerUp` with these odds: 6% life, 8% bomb, 20% weapon. They are applied in `applyPowerUp`. Weapon pickups raise `player.level` up to 3 (which changes the spread in `fireWeapon`), then toggle between `vulcan` and `laser`.
- **Best score** is stored in `localStorage` under `skyfighter.bestScore`.

### localStorage keys

These are per-browser and must keep their names, or saved player settings will be lost:

- `skyfighter.bestScore`
- `skyfighter.muted`
- `skyfighter.difficulty`
- `skyfighter.reducedMotion`

## Conventions

- **Single file.** Keep HTML, CSS, and JS in `index.html` unless the user asks for a split. Do not add a bundler or framework.
- **No dependencies.** Use only browser built-ins (Canvas 2D, Web Audio, `localStorage`). Do not add CDN links without a request.
- **Style matches the existing code.** Use 2-space indentation, single quotes in JS, `const` by default, arrow functions for callbacks, and short section comments in the form `// ---------- Section ----------`.
- **Keep the IIFE closed.** New state and helpers belong inside the existing `(function () { ... })()` so nothing leaks onto `window`.
- **Game-loop code:** `update(dt)` and `draw()` run every frame. Avoid allocating new objects or arrays in hot paths where a reused structure would do, and cap `dt` as `loop` already does (`Math.min(dt, 0.05)`).
- **Accessibility:** announce important state changes through `announce()` (which writes to `#srLive`). Respect `reducedMotion` when adding screen shake, flashes, or particle effects.
- **Touch and desktop:** any new control needs both a keyboard path and a touch path. Use `e.preventDefault()` with `{ passive: false }` on touch handlers that must stop scrolling.

## Working on this repo

- **Verify in a browser.** There are no automated tests or linters. After a change, open `index.html` in a browser (or use a headless Chromium with Playwright) and play through start, combat, pause, game over, and restart. Check the console for errors.
- **Syntax check.** For a quick check of the script without a browser, extract the `<script>` block and run `node --check` on it.
- **Git.** Develop on the branch assigned to the session. Use small, descriptive commit messages in the imperative mood (existing examples: "Add Sky Fighter: Raiden-style vertical shoot-em-up game").
- **Do not commit generated or local files.** The repo currently contains only `README.md` and `index.html`.

## Known gaps

- `README.md` is only a title and a timestamp. It does not describe the game. Expand it if the user asks for user-facing docs.
- There are no tests, so balance changes (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`) are checked only by playing.
