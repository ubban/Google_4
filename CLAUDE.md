# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up that runs in the browser. The whole game lives in one self-contained file:

- `index.html` — inline CSS (`<style>`, starts ~line 10), markup (`<div id="container">`, ~line 362), and a single IIFE `<script>` (~line 422) containing all game logic.
- `README.md` — placeholder; not a source of truth for the game.

There is no build step, no bundler, no `package.json`, no test suite, and no linter. Do not introduce any of these unless asked.

## Running and verifying

- Open `index.html` directly in a browser, or serve the directory with any static server (e.g. `python3 -m http.server`).
- Verification is manual: play the game in a browser and check the behaviour you changed. Use the in-browser console to catch runtime errors.
- Test both desktop (keyboard/mouse) and touch layouts when touching input, HUD or layout code. The layout is designed for a 480×720 logical canvas (`BASE_W`, `BASE_H`), scaled by `resize()`.

## Code layout inside `index.html`

The script is one IIFE with `'use strict'`. Roughly in order:

1. **DOM refs** — `document.getElementById(...)` lookups at the top of the IIFE.
2. **Persistence keys** — `STORAGE_BEST`, `STORAGE_MUTE`, `STORAGE_DIFF`, `STORAGE_REDUCED` (all prefixed `skyfighter.`).
3. **Adaptive performance** — `trackPerf()` steps a `quality` level (0 full / 1 medium / 2 low) based on average frame time.
4. **Tuning constants** — `DIFFICULTIES` (spawn rates, enemy speed, shoot chance, lives, bombs, boss HP multiplier), `KILLS_PER_STAGE`, `THEMES` (per-stage sky palettes), `ENEMY_TYPES` (grunt, interceptor, tank), `BOSSES` (six bosses, cycled by stage).
5. **Audio** — `ensureAudio()`, `beep()`, `noiseBurst()`, and a named `sfx` table. Audio is created lazily on the first user gesture.
6. **HUD helpers** — `renderHearts`, `renderBombs`, `flashHit`, `showComboPop`, `showStageBanner`, `screenShake`, `announce` (screen-reader live region).
7. **Input** — keyboard (`keys` map, keyed by `e.code`), mouse/canvas steering, touch joystick and FIRE/BOMB buttons.
8. **Game state** — `resetGame()` initialises all mutable state (`score`, `lives`, `maxLives`, `running`, `paused`, `gameOver`, etc.).
9. **Entities and systems** — `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`, `spawnExplosion`, `addScore` / combo chain (`breakChain`), `useBomb`, `fireWeapon`, `applyPowerUp`.
10. **Boss patterns** — `fireRadial`, `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`, `fireCross`, `fireHomingSeeded`, `fireDoubleRing`, collected in `BOSS_PATTERNS`; driven by `updateBoss()`.
11. **Flow** — `loseLife()`, `killBoss()`, `endGame()`, `togglePause()`.
12. **Rendering** — `drawPlane`, `drawBossShip`, `drawClouds`, `draw()`.
13. **Main loop** — `loop(now)` clamps `dt` to 0.05 s, then calls `trackPerf`, `update(dt)`, and `draw()` while running and not paused. Ends by starting `requestAnimationFrame(loop)`.

Game logic is updated in `update(dt)`; rendering happens only in `draw()`. Keep that separation.

## Gameplay facts to preserve

- Controls: mouse move to steer / click to fire; Arrows or WASD to move; Space to fire; X or Shift for bomb; P or Esc to pause. Touch: drag left zone to fly, FIRE and BOMB buttons.
- Three difficulties: easy, normal (default), hard.
- Weapons upgrade through power-ups up to level 3 (`applyPowerUp`).
- Power-up drop chances are set in `maybeSpawnPowerUp`: life (6%), bomb (8%), weapon (20%).
- Score is multiplied by a combo chain: multiplier = `1 + min(9, floor(chain / 5))`. Each kill refreshes a 2.2 s chain timer; the chain breaks when the timer runs out or a life is lost (`breakChain`).
- Best score is saved to `localStorage` under `skyfighter.bestScore`.

## Conventions

- Keep everything in `index.html` unless the user asks for a split. Do not add external dependencies or CDN links without asking.
- Match the existing style: 2-space indentation, single quotes in JS, `const`/`let` (no `var`), small named functions, and `// ---------- Section ----------` banners to separate sections.
- Balance changes belong in the tuning constants (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `THEMES`) rather than hard-coded in logic.
- Respect the reduced-motion setting: effects that shake or flash should check `reducedMotion` / the `reduced-motion` body class.
- Keep the touch path working. Any new interactive control needs a touch equivalent and must not rely on hover.
- Keep accessibility hooks intact: `srLive` announcements for important events, and `aria-*` attributes on controls.
- Do not persist new state under keys other than the `skyfighter.*` prefix.

## Git

- Development branch for this work: `claude/gifted-tesla-qkwiix`. Do not push to other branches without explicit permission.
- Do not open a pull request unless the user asks for one.
