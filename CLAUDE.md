# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo contains **Sky Fighter**, a single-page HTML5 canvas vertical shoot-'em-up (Raiden-style). The entire game lives in one file, `index.html`: inline `<style>`, markup, and one inline `<script>` IIFE. There is no build step, no bundler, no package manager, and no test suite.

Other files:
- `README.md` — minimal placeholder (project title only).
- `.git/` — the only history: `Initial commit` and `Add Sky Fighter ...`.

## Running the game

- Open `index.html` directly in a browser, or serve the directory with any static server (e.g. `python3 -m http.server`) and visit it.
- No install step is required.

## Code layout (`index.html`)

Rough map of the inline script, in file order:

- **Setup & DOM refs** (~line 422): `canvas`, HUD elements, overlay, touch controls, all `getElementById` lookups at the top of the IIFE.
- **Constants** (~lines 463–530): storage keys (`skyfighter.*`), `DIFFICULTIES` (easy/normal/hard tuning table), `KILLS_PER_STAGE`, `THEMES` (per-stage sky palettes), `ENEMY_TYPES`, `BOSSES`.
- **Audio** (~lines 545–680): lazy `ensureAudio()` on first user gesture, `beep()` and `noiseBurst()` synth helpers, `EXPLODE_SFX` map.
- **HUD & feedback helpers** (~lines 680–750): hearts/bombs rendering, hit flash, combo pops, stage banner, screen shake, mute and difficulty toggles.
- **Layout & input** (~lines 753–870): `resize()` scales the `BASE_W × BASE_H` (480×720) logical canvas to the viewport; joystick and pointer handling for touch; keyboard handlers are at ~line 769.
- **Game state** (~lines 875–912): `resetGame()` initializes all mutable state.
- **Rendering of entities** (~lines 913–1070, 1507–1640): `drawPlane`, `drawBossShip`, `draw()`, cloud/parallax layers.
- **Spawning & combat** (~lines 1022–1150): `spawnEnemy`, `spawnBoss`, power-up drops, explosions, scoring/kill chain, bombs, `fireWeapon()`.
- **Simulation** (~lines 1148–1305): `update(dt)` — the per-frame logic for entities, collisions, and stage progression.
- **Bosses** (~lines 1324–1470): bullet pattern generators (`fireRadial`, `fireSpiralBurst`, `fireHomingSeeded`, …), `BOSS_PATTERNS`, `updateBoss`, `killBoss`.
- **Game flow** (~lines 1456–1506): `loseLife`, `endGame`, `togglePause`.
- **Main loop** (~line 1640): `loop(now)` uses `requestAnimationFrame`, clamps `dt` to 50 ms, and runs `update` + `draw` only when running and not paused.

### Key conventions in the code

- Everything is inside one IIFE with `'use strict'`; nothing is exported to `window`. Keep new code inside that scope.
- Game logic is framerate-independent: use `dt` (seconds) for movement and timers, never per-frame counts.
- Logical coordinates are always 480×720; rendering is scaled by `resize()`. Don't hard-code screen pixel sizes.
- Persistence uses `localStorage` with the `skyfighter.*` key prefix. Read/write through the existing `STORAGE_*` constants so keys stay in one place.
- Adaptive quality: `trackPerf()` lowers `quality` when average frame time is slow and raises it when fast. Respect this when adding expensive effects.
- Reduced-motion is a user setting (`reducedMotion`); effects such as screen shake should honor it.
- Audio is only started after a user gesture (`ensureAudio()`); call it from input handlers, not at load time.

## Controls (for reference when changing input)

- Keyboard: arrows/WASD move, Space fires, X or Shift bombs, P or Esc pauses.
- Mouse/pointer: move to steer, click to fire.
- Touch: left drag-zone joystick, FIRE button, BOMB button.

## Making changes

- Keep the single-file structure unless the user asks otherwise. Don't add a build system or dependencies for small changes.
- Preserve the existing style: 2-space indentation, `const`/`let`, arrow functions for handlers, and short helper functions.
- Tune gameplay numbers in the `DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, and `BOSS_PATTERNS` tables rather than scattering magic numbers.
- Test by loading the page in a browser and playing: check title/overlay, a full stage through to a boss, pause/resume, game over, and best-score persistence after reload. Also check touch layout at phone width when touching HUD or controls.
- There are no automated tests or linters configured. If you add any, document the command here.

## Git

- Development branch for this work: `claude/gifted-tesla-6mt7cc`. Commit and push there; don't push to other branches without explicit permission.
- Don't open pull requests unless the user asks.
