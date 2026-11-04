# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Google_4 is a single-page browser game, **Sky Fighter**: a Raiden-style vertical shoot-'em-up. The entire game lives in one file, `index.html`, with inline `<style>` and one inline `<script>` (an IIFE). There is no build step, no package manager, no test suite, and no external assets; audio is synthesized with WebAudio.

Files:
- `index.html` — the whole game (markup, CSS, JS).
- `README.md` — minimal project title.
- `CLAUDE.md` — this file.

## Running it

Open `index.html` in a browser, or serve the directory with any static server (e.g. `python3 -m http.server`). There is nothing to install or compile.

## Code layout inside `index.html`

The script is a single `(function () { 'use strict'; ... })();` closure. Major sections, in source order (approximate line numbers drift; search by name):

- **DOM references and storage keys** — `canvas`, HUD elements, and `STORAGE_*` keys (`skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion`).
- **Adaptive performance** — `trackPerf` samples frame time and steps a `quality` level (0–2) up or down.
- **Config tables** — `DIFFICULTIES` (easy/normal/hard: spawn rates, enemy speed, shoot chance, lives, bombs, boss HP multiplier), `KILLS_PER_STAGE`, `THEMES` (per-stage sky palettes, cycled by stage), `ENEMY_TYPES` (grunt, interceptor, tank), `BOSSES` (six bosses: VANGUARD, BEHEMOTH, SPECTER, DREADNOUGHT, WRAITH, TITAN).
- **Audio** — `ensureAudio`, `beep`, `noiseBurst`, and the `sfx` object. Audio is created lazily on first user gesture.
- **HUD and feedback** — `renderHearts`, `renderBombs`, `flashHit`, `showComboPop`, `showStageBanner`, `screenShake`, `announce` (screen-reader live region).
- **Canvas sizing** — `BASE_W = 480`, `BASE_H = 720`; `resize()` scales the canvas to fit the viewport.
- **Input** — keyboard (`keys` map keyed by `e.code`), touch joystick (`joystickStart/Move/End`), fire and bomb buttons, and mouse steering (`canvasPointFromEvent`). Keyboard/joystick takes priority over mouse steering.
- **Game state** — `resetGame()` initializes `player`, arrays (`bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`), `boss`, score, lives, bombs, stage, and chain/multiplier.
- **Drawing** — `drawPlane`, `drawBossShip`, `drawClouds`, and `draw()`.
- **Spawning and combat** — `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`, `spawnExplosion`, `addScore` / `breakChain` (combo multiplier), `useBomb`, `fireWeapon`.
- **Update loop** — `update(dt)` advances movement, bullets, enemies, collisions, and power-ups. `updateBoss(dt)` runs boss phases; `BOSS_PATTERNS` and the `fire*` helpers (radial, aimed spread, columns, spiral, cross, homing, double ring) define boss attack patterns.
- **Lifecycle** — `loseLife()`, `killBoss()`, `endGame()` (persists best score), `togglePause()`.
- **Main loop** — `loop(now)` clamps `dt` to 50 ms, then calls `update` and `draw` while running and not paused.

## Game rules worth knowing

- Power-ups (`life`, `bomb`, `weapon`) drop from enemies; `applyPowerUp` handles them. Weapon pickups upgrade the current weapon to level 3, then toggle between `vulcan` and `laser` at level 1.
- Each stage ends with a boss: once `stageKills` reaches `KILLS_PER_STAGE` (18), a boss is spawned after a short delay. Defeating it advances `stage`, and the background theme is `THEMES[(stage - 1) % THEMES.length]`.
- Keys: arrows/WASD move, Space fires, X / Shift bombs, P / Escape pauses.
- Best score is stored in `localStorage` and updated in `endGame()`.

## Conventions

- Keep the game in a single `index.html` unless the user asks for a restructure. Do not add a build system, bundler, or framework without being asked.
- Stay in the existing style: `const`/`let`, arrow functions for callbacks, semicolons on statements, 2-space indentation, and `// ---------- Section ----------` banner comments to separate sections.
- Tune gameplay through the config tables (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `THEMES`) rather than hard-coding numbers in logic.
- Any new storage must use a `skyfighter.*` key and be wrapped so it still works if `localStorage` is unavailable or throws.
- Keep the layout usable on touch devices and small viewports: the `viewport` meta disables zoom, and touch controls appear only when a touch device is detected.
- Respect `prefers-reduced-motion` (the `reducedMotion` flag and `reduced-motion` body class) when adding screen shake, flashes, or other motion effects.
- Keep accessibility hooks intact: announce important state changes through `announce()` / the `srLive` region.

## Verifying changes

There are no automated tests or linters configured. To verify a change:
1. Load `index.html` in a browser and start a game from the overlay.
2. Exercise the area you changed (movement, firing, a boss fight, pause, game over, difficulty switch, mute and reduced-motion toggles).
3. Check the browser console for errors.
4. For touch-related changes, test with device emulation or a touch device.

If you add tests or tooling later, document the commands here.

## Git

- Develop on the branch named in the session instructions; do not push to other branches without explicit permission.
- Write descriptive commit messages in the imperative mood.
