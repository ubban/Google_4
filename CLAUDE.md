# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a single-file browser game, **Sky Fighter**: a Raiden-style vertical shoot-'em-up. There is no build system, package manager, test suite, or server-side code.

Files:

- `index.html` — the entire game: markup (`<body>`), CSS (`<style>`), and JavaScript (one `<script>` block wrapped in an IIFE with `'use strict'`).
- `README.md` — minimal project readme.

## Running

Open `index.html` directly in a browser. No install or build step is required. Any static file server also works.

## Code layout (`index.html`)

Approximate line ranges; they drift as the file changes, so search by name rather than trusting numbers.

- **Markup and styles** (~lines 1–420): HUD, overlay/menu, touch controls, boss bar, and a `#srLive` live region for screen readers.
- **Setup and state** (~422–530): DOM references, `STORAGE_*` keys, `DIFFICULTIES`, `THEMES`, `ENEMY_TYPES`, `BOSSES`.
- **Adaptive quality** (`trackPerf`, `quality`): samples frame times and steps quality 0–2 when the frame rate drops or recovers.
- **Audio** (`ensureAudio`, `beep`, `noiseBurst`, `sfx`): Web Audio synthesis only. No audio files are loaded.
- **HUD helpers** (`renderHearts`, `renderBombs`, `showComboPop`, `showStageBanner`, `screenShake`, `flashHit`).
- **Input** (~769–865): keyboard, touch joystick and fire/bomb buttons, and mouse aiming on the canvas.
- **Game state** (`resetGame`, ~875): `score`, `lives`, `maxLives`, `running`, `paused`, `gameOver`, plus player, enemies, bullets, and power-ups.
- **Drawing** (`drawPlane`, `drawBossShip`, `drawClouds`, `draw`, ~913–1640): all rendering is on a 2D canvas.
- **Spawning and combat** (`spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`, `fireWeapon`, `useBomb`, `addScore`, `breakChain`).
- **Update loop** (`update(dt)`, ~1148): per-frame simulation, with `dt` clamped to 50 ms.
- **Power-ups** (`applyPowerUp`): `life`, `bomb`, and `weapon`. Weapon upgrades go through three levels, then toggle between `vulcan` and `laser`.
- **Bosses** (`BOSS_PATTERNS`, `updateBoss`, `killBoss`): each boss has a bullet pattern function, such as radial, aimed spread, columns, spiral, cross, homing, or double ring.
- **Game flow** (`loseLife`, `endGame`, `togglePause`, `loop`, `startBtn` handler).

## Game rules and tuning

- Canvas base resolution is `480 x 720`. `resize()` scales it to the window, capped at 1.6x.
- Difficulty (`easy` / `normal` / `hard`) sets spawn rates, enemy speed, enemy shoot chance, starting lives and bombs, and boss HP multiplier. Tuning lives in `DIFFICULTIES`.
- After `KILLS_PER_STAGE` (18) kills in a stage, a boss spawns. Stage N uses `THEMES[(N-1) % 6]` and `BOSSES[(N-1) % 6]`, so both cycle.
- Enemy types: `grunt`, `interceptor` (weaves), and `tank` (4 HP, unlocks at stage 2).
- Combo: each kill extends a 2.2 s chain window. The multiplier is `1 + min(9, floor(chain / 5))`. Letting the window lapse calls `breakChain()`.

## Persistence

`localStorage` keys, all prefixed `skyfighter.`:

- `bestScore`, `muted`, `difficulty`, `reducedMotion`

Every access should be safe to fail. The game currently reads and writes these keys directly, so keep new storage code consistent with that pattern.

## Controls

- **Move:** Arrow keys or WASD, or drag the touch joystick.
- **Fire:** Hold Space, or use the touch fire button or mouse button.
- **Bomb:** `X` or either Shift, or the bomb button.
- **Pause:** `P` or `Escape`.

## Conventions

- Keep everything in `index.html` unless the user asks for a split. Do not add a build step, framework, or external dependencies without a clear reason.
- Match the existing style: 2-space indentation, single quotes in JavaScript, `const`/`let`, arrow functions for callbacks, and short section comments such as `// ---------- Input ----------`.
- Use the existing helpers (`announce`, `sfx.*`, `renderHearts`, and so on) rather than duplicating them.
- Keep changes small and local. The game loop runs every frame, so avoid per-frame allocations in hot paths such as `update` and `draw`.
- Preserve accessibility hooks: the `srLive` announcements, the `reducedMotion` class, and keyboard reachability of the menu buttons.

## Verification

There are no automated tests or linters. To verify a change:

1. Open `index.html` in a browser (or serve the directory with any static server).
2. Start a run and check the affected behavior, including at least one difficulty level.
3. Check the touch layout at phone width if the change touches HUD or controls.
4. Check the browser console for errors.

## Git

- Develop on the branch the task specifies. Do not push to other branches without permission.
- Do not open pull requests unless the user asks.
