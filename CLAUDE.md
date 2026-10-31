# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Google_4 contains **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs in the browser. The whole game is a single self-contained file:

- `index.html` (~1,670 lines): inline `<style>`, the DOM HUD and overlays, and one inline `<script>` IIFE that holds all game logic.
- `README.md`: a one-line project title.

There is no build step, no bundler, no package manager, no dependencies, and no test suite. Nothing is fetched from a CDN.

## Running and verifying

- Open `index.html` directly in a browser, or serve the directory with any static server (for example `python3 -m http.server`).
- Verification is manual: play the game in a browser and check the behaviour you changed. For changes to rendering, input, or HUD, use a real browser rather than relying on reading the code.
- Check the browser console for errors after any change.

## Code layout inside `index.html`

The script is one IIFE with `'use strict'`. Sections, roughly in source order:

1. **DOM references**: `const xxxEl = document.getElementById(...)` at the top. Add new element lookups here.
2. **Storage keys**: `STORAGE_BEST`, `STORAGE_MUTE`, `STORAGE_DIFF`, `STORAGE_REDUCED`, all under the `skyfighter.` prefix.
3. **Adaptive performance**: `trackPerf` samples frame time and steps `quality` (0 full, 1 medium, 2 low).
4. **Config tables**: `DIFFICULTIES`, `THEMES` (per-stage sky palettes), `ENEMY_TYPES` (grunt, interceptor, tank), `BOSSES` (six bosses, picked by `BOSSES[(stage - 1) % BOSSES.length]`; boss HP scales with stage and difficulty), `KILLS_PER_STAGE = 18`.
5. **Persistence and settings**: best score, mute, difficulty, and reduced motion, all read from and written to `localStorage`.
6. **Audio**: a lazily created `AudioContext` (`ensureAudio`), with `beep` and `noiseBurst` primitives and a `sfx` table built on them.
7. **HUD helpers**: `renderHearts`, `renderBombs`, `flashHit`, `showComboPop`, `showStageBanner`, `screenShake`, `setDifficulty`.
8. **Canvas sizing and input**: `resize()` fits a `BASE_W × BASE_H` (480 × 720) logical canvas. Keyboard, mouse, and touch (joystick and buttons) handlers live here.
9. **Game state**: globals such as `score`, `lives`, `bombCount`, `stage`, `chain`, `multiplier`, plus the entity arrays `bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`, and `boss`.
10. **Gameplay**: `resetGame`, `spawnEnemy`, `spawnBoss`, `fireWeapon`, `useBomb`, `applyPowerUp`, `updateBoss`, `killBoss`, `loseLife`, `endGame`, `togglePause`.
11. **Boss attack patterns**: `BOSS_PATTERNS` and the `fireRadial`, `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`, `fireCross`, `fireHomingSeeded`, and `fireDoubleRing` helpers.
12. **Rendering**: `drawPlane`, `drawBossShip`, `drawClouds`, and `draw()`.
13. **Main loop**: `loop(now)` clamps `dt` to 50 ms, calls `trackPerf`, then `update(dt)` and `draw()` while the game is running and not paused.

Entities are plain objects with `x`, `y`, `w`, `h`, and per-type fields. Collision uses `rectsOverlap`. Removal happens by iterating backwards over arrays and splicing.

## Controls and game rules (for reference when changing behaviour)

- **Move**: arrow keys or WASD, mouse, or a touch joystick.
- **Fire**: Space, hold the mouse button, or the FIRE button on touch.
- **Bomb**: X or Shift, or the BOMB button on touch.
- **Pause**: P or Escape.
- **Power-ups** (`applyPowerUp`): `life` (+1 heart, capped at max), `bomb` (+1 bomb, capped), `weapon` (raises weapon level 1 to 3, then toggles between `vulcan` and `laser`).
- **Difficulty** (`DIFFICULTIES`): easy, normal, and hard differ in spawn rate, enemy speed, shoot chance, starting lives and bombs, and boss HP multiplier.
- **Stages**: clearing `KILLS_PER_STAGE` kills triggers a boss. Each stage advances `stage` and switches the theme.

## Conventions

- Keep everything in `index.html` unless the user asks for a split. Do not introduce a build step or external dependencies without asking.
- Match the existing style: two-space indentation, single quotes in JS, `const` by default, arrow functions for callbacks, and short section comments in the `// ---------- Name ----------` form.
- Persist new user settings with a `skyfighter.`-prefixed key, and wrap `localStorage` use in the same simple pattern the file already uses.
- Keep the accessibility hooks working: `announce()` feeds the `srLive` live region, and reduced-motion must still suppress shake and flash effects.
- Keep touch controls working alongside keyboard and mouse. Test changes to input on both paths.
- Keep the HUD text and overlay copy in sync with the controls hint in `#controlsHint`.

## Git workflow

- Develop on the branch assigned for the task (currently `claude/gifted-tesla-56pbkn`). Do not push to `main` without explicit permission.
- Commit with clear, descriptive messages. Push with `git push -u origin <branch>`.
- Do not open pull requests unless the user asks for one.
