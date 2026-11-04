# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up that runs in the browser. The entire game is a single self-contained file, `index.html` (~1,670 lines): inline CSS in `<style>`, markup, and one inline `<script>` IIFE. There is no build step, no package manager, no bundler, no framework, no test suite, and no external dependencies (no CDN scripts). `README.md` is a placeholder.

To run it, open `index.html` in a browser (or serve the directory with any static server). Nothing needs to be installed.

## Repository layout

```
index.html   # The whole game: CSS, HTML, and JS
README.md    # Placeholder
```

Keep the game in one file unless the user asks for a split. Don't add a build system, package.json, or dependencies without an explicit request.

## Architecture of `index.html`

Ordered top to bottom:

1. **`<head>`** – viewport meta (locks zoom, `viewport-fit=cover` for notched phones), theme color, and an inline SVG favicon.
2. **`<style>`** (around lines 10–359) – dark color scheme. HUD elements (`#hud`, `.stat`, `#hearts`, `#bombs`), boss bar (`#bossBarWrap`), vignette/flash overlays, touch controls, overlay card, and keyframe animations.
3. **`<body>`** (around lines 361–421) – `#container` holding the `#gameCanvas`, HUD, boss bar, combo pop, stage banner, touch controls (`#moveZone` joystick, `#actionZone` fire/bomb), and the start/pause/game-over `#overlay` with difficulty buttons (`.diffBtn`).
4. **`<script>`** (around lines 422–1668) – a single IIFE, `(function () { ... })();`, containing all logic. Sections, in order:
   - **DOM refs and storage keys** – `STORAGE_BEST`, `STORAGE_MUTE`, `STORAGE_DIFF`, `STORAGE_REDUCED` (all `skyfighter.*` in `localStorage`).
   - **Adaptive quality** – `trackPerf()` samples frame time over 45 frames and steps `quality` (0 full, 1 medium, 2 low) to keep frame rate up.
   - **Tuning data** – `DIFFICULTIES` (spawn rate, enemy speed, shoot chance, lives, bombs, boss HP multiplier), `KILLS_PER_STAGE`, `THEMES` (six sky palettes cycled per stage), `ENEMY_TYPES` (grunt, interceptor, tank), `BOSSES` (six bosses, each with its own colors and HP multiplier).
   - **Audio** – lazily created `AudioContext` via `ensureAudio()`; procedural sound via `beep()` and `noiseBurst()`, with named effects in `sfx`. Muting is persisted.
   - **UI helpers** – `renderHearts`, `renderBombs`, `flashHit`, `flashBomb`, `showComboPop`, `showStageBanner`, `screenShake`, `announce` (screen-reader live region).
   - **Scaling** – `BASE_W = 480`, `BASE_H = 720`; `resize()` scales the canvas to fit the window, capped at 1.6×.
   - **Input** – keyboard (`keys[]`), touch joystick and buttons, and mouse/pointer targeting. See Controls below.
   - **Game state** – module-level `let`s: `player`, `bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`, `boss`, `score`, `lives`, `stage`, `chain`, `multiplier`, and others. `resetGame()` reinitializes them.
   - **Drawing** – `drawPlane`, `drawBossShip`, `drawClouds`, and `draw()`.
   - **Simulation** – `update(dt)` (main per-frame logic), `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`, `applyPowerUp`, `useBomb`, `fireWeapon`, the boss attack patterns (`fireRadial`, `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`, `fireCross`, `fireHomingSeeded`, `fireDoubleRing`), `updateBoss`, `killBoss`, `loseLife`, `endGame`, `togglePause`.
   - **Loop** – `loop(now)` runs on `requestAnimationFrame`, clamps `dt` to 50 ms, and calls `update` + `draw` only when `running && !paused`. The start button is wired at the bottom.

### Game rules worth knowing

- Stages advance after `KILLS_PER_STAGE` (18) kills, then a boss fights. Themes cycle with stage.
- Enemy types: `grunt` (1 HP), `interceptor` (weaves, fast), `tank` (4 HP, from stage 2).
- Power-ups: `life`, `bomb`, `weapon` (raises weapon level up to 3, or toggles vulcan/laser at max).
- Chain kills build a score `multiplier`; `breakChain()` resets it.
- Best score is persisted to `localStorage` under `skyfighter.bestScore`.

### Controls

- Move: arrow keys / WASD (via `keys[]`), touch joystick (drag left side), or mouse movement over the canvas.
- Fire: Space, mouse click/hold, or the touch FIRE button (`firing` flag).
- Bomb: `X` or Shift (also the bomb button on touch).
- Pause: `P` or `Escape`.

## Conventions

- **Plain JS, ES2017+ syntax, no modules.** Match the existing style: 2-space indent, single quotes, semicolons, `const` by default, `let` for mutable game state.
- **Keep the IIFE.** Don't leak new globals; put new state and helpers inside the IIFE.
- **Tuning belongs in the data tables** (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `THEMES`) rather than hard-coded magic numbers in logic, when practical.
- **Frame-rate independence:** time-based movement uses `dt` (seconds). Preserve that when adding entities.
- **Persistence:** all `localStorage` access is with the `skyfighter.*` keys. Keep the existing `Number(...) || default` pattern for reads.
- **Accessibility:** respect `prefers-reduced-motion` (`reducedMotion` flag) for screen shake and flashes; announce important state changes through `announce()`.
- **Mobile:** keep the layout working at phone width, with touch controls and `touch-action: none` on the game surface. Don't add page scroll.
- **Canvas coordinates:** logic uses the 480×720 base space; the canvas is scaled for display.

## Working on this project

- There are no automated tests or linters. Verify changes by loading `index.html` in a browser and playing through the affected path (start, fire, bomb, pause, a stage transition, a boss, game over, and difficulty switching). Check the browser console for errors.
- Check both desktop (keyboard) and touch layouts when you change HUD, overlay, or control code.
- Keep edits minimal and local. Avoid reformatting unrelated parts of the file, since the diff should stay easy to review.
- Don't commit stray files (editor backups, screenshots, local notes) to the repo.

## Git

- Development branch for this work: `claude/gifted-tesla-j62h1l`. Push with `git push -u origin <branch>`.
- Commit messages: short imperative summary line, with a body when the change needs explaining.
- Don't open a pull request unless the user asks for one.
