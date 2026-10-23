# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Google_4 is a static, dependency-free browser game: **Sky Fighter**, a Raiden-style vertical shoot-'em-up rendered on a single HTML5 `<canvas>`.

The whole game lives in one file, `index.html`:
- `<style>` (~lines 10–420): CSS for the page, HUD, overlays, touch controls.
- `<body>` markup (~lines 360–420): HUD elements, overlay cards, touch controls, the canvas.
- One inline `<script>` IIFE (~lines 422 to end): all game logic, rendering, audio, input.

Other files:
- `README.md`: one-line placeholder title. Keep it short.

There is no build step, package manager, test suite, linter, or server-side component. Do not add one unless asked.

## Running and verifying

- Open `index.html` directly in a browser, or serve the directory with any static server (e.g. `python3 -m http.server`).
- Verify changes by playing in a browser. Check desktop keyboard, touch controls (narrow viewport), audio toggle, pause, and the difficulty buttons.
- For gameplay changes, exercise both the early stages (grunts/interceptors) and a boss fight (stage-end at 18 kills per stage).

## Code structure (inside the script IIFE)

Roughly in order of appearance:

1. **DOM references**: `const xEl = document.getElementById(...)`.
2. **Storage keys**: `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion` (all `localStorage`).
3. **Adaptive quality**: `trackPerf()` samples frame times and steps `quality` 0–2 when FPS drops.
4. **Config tables**:
   - `DIFFICULTIES`: spawn rate, enemy speed, shoot chance, lives, bombs, boss HP multiplier. Easy/Normal/Hard.
   - `KILLS_PER_STAGE = 18`.
   - `THEMES`: sky gradients, cloud/sun colors, per stage (cycled).
   - `ENEMY_TYPES`: `grunt`, `interceptor` (weaving), `tank` (4 HP, stage 2+).
   - `BOSSES`: six bosses with colors and `hpMul`, cycled per stage.
5. **Audio**: `ensureAudio()` lazily creates the `AudioContext` on first user gesture. `beep()`, `noiseBurst()`, and a `sfx` object build sounds procedurally. Nothing is loaded from disk.
6. **HUD helpers**: `renderHearts`, `renderBombs`, `flashHit`, `flashBomb`, `showComboPop`, `showStageBanner`, `screenShake`, `announce` (screen-reader live region).
7. **Layout**: `resize()` fits a `BASE_W × BASE_H` (480×720) logical canvas to the viewport.
8. **Input**:
   - Keyboard: arrows/WASD move; Space or hold fire; `KeyX`/Shift = bomb; `KeyP`/Escape = pause.
   - Mouse: canvas drag moves the plane; `fireBtn` mousedown fires.
   - Touch: virtual joystick (`moveZone`), fire and bomb buttons.
9. **Game state**: module-level `let`s, e.g. `player, bullets, enemies, enemyBullets, particles, powerUps, boss`, plus `score, lives, maxLives, running, paused, gameOver`. `resetGame()` reinitializes them.
10. **Update/draw**:
    - `update(dt)` runs game logic. `draw()` renders. `loop(now)` drives both via `requestAnimationFrame`, with `dt` clamped to 0.05 s.
    - Drawing helpers: `drawPlane`, `drawBossShip`, `drawClouds`.
11. **Weapons and power-ups**: `fireWeapon()` (Vulcan/Laser, 3 levels), `useBomb()` (clears screen, damages bosses), `applyPowerUp(type)` for `life`, `bomb`, `weapon`.
12. **Bosses**: `spawnBoss`, `updateBoss`, `killBoss`, and a `BOSS_PATTERNS` array of bullet patterns (radial, aimed spread, columns, spiral, cross, homing, double ring).
13. **Session flow**: `loseLife`, `endGame`, `togglePause`, and the start button handler.

## Conventions

- **Single file, no dependencies.** Keep everything in `index.html`. Do not add frameworks, bundlers, or CDN scripts without asking.
- **Match the existing style.** 2-space indent, single quotes in JS, `const` by default, arrow functions for callbacks, section banners like `// ---------- Input ----------`.
- **Tuning lives in config tables.** Prefer adjusting `DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `THEMES` over hard-coding numbers in logic.
- **Logical coordinates.** Game logic uses the 480×720 base space; `resize()` handles scaling. Don't use raw CSS pixels in gameplay code.
- **Respect reduced motion.** The `reducedMotion` flag (persisted, defaults to the OS `prefers-reduced-motion` setting) scales down effects. `screenShake()` already does this; new visual effects should too. The `reduced-motion` body class is available for CSS.
- **Keep it accessible.** Announce important state changes via `announce()`; keep ARIA labels on buttons.
- **Mobile first.** Touch handlers use `{ passive: false }` where they call `preventDefault()`. Don't break the touch layout when changing HUD or overlay markup.
- **Persist with the `skyfighter.*` keys only**, and wrap storage reads in a fallback so the game still runs if `localStorage` is unavailable.
- Audio must start from a user gesture (`ensureAudio()`), since browsers block autoplay.

## Making changes

- Read the relevant section of `index.html` before editing; line numbers drift, so search by function name.
- Keep diffs focused. Don't reformat unrelated code.
- After editing, play-test the affected feature in a browser before reporting it done. If you cannot run a browser, say so explicitly rather than claiming it works.
- Commit with clear messages describing the gameplay or UI change.

## Git

- Develop on the branch the task specifies (currently `claude/gifted-tesla-5b49fe`). Don't push to other branches without permission.
- Don't open a pull request unless explicitly asked.
