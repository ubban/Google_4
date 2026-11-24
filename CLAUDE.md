# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a small, dependency-free browser game: **Sky Fighter**, a Raiden-style vertical shoot-'em-up rendered on an HTML5 `<canvas>`. The whole game is a single file, `index.html` (~1,670 lines: inline `<style>`, markup, and one inline `<script>` IIFE). `README.md` is a placeholder and has no setup instructions.

## Repository layout

```
.
├── index.html   # The entire game: CSS, DOM/HUD markup, and JS game logic
├── README.md    # Placeholder
└── CLAUDE.md    # This file
```

There is no `package.json`, bundler, linter config, test suite, or CI workflow.

## Running the game

No build or install step is needed. Open `index.html` directly in a browser, or serve the directory with any static server (for example `python3 -m http.server` and visit `http://localhost:8000`).

## Architecture of `index.html`

**Markup (lines ~3–360 are CSS, ~361–421 are DOM).** Key element IDs the script depends on: `gameCanvas`, `overlay`, `startBtn`, `hearts`, `bombs`, `score`, `best`, `mult`, `weaponName`, `weaponLvl`, `bossBarWrap`, `bossBarFill`, `bossName`, `comboPop`, `stageBanner`, `muteBtn`, `motionBtn`, `pauseBtn`, `touchControls`, `moveZone`, `fireBtn`, `bombBtn`, `srLive`. Renaming any of these breaks `getElementById` lookups at startup.

**Script (lines ~422–1668).** Everything lives in one `(function () { 'use strict'; ... })()` closure, so nothing is exposed on `window`. Sections, in order:

1. **DOM refs and constants**: element lookups, `STORAGE_*` keys, `DIFFICULTIES`, `KILLS_PER_STAGE` (18), `THEMES` (six sky palettes that cycle by stage), `ENEMY_TYPES` (`grunt`, `interceptor`, `tank`), `BOSSES` (six boss designs, indexed by `(stage - 1) % 6`).
2. **Adaptive quality** (`trackPerf`): averages the last 45 frame times and steps `quality` 0→2 when frames run slow (<40 fps) and back when fast (>56 fps). Drawing code reads `quality` to reduce effects.
3. **Persistence**: `localStorage` keys `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion`.
4. **Audio**: a lazily created Web Audio context (`ensureAudio`) with `beep` / `noiseBurst` primitives and a `sfx` object. Audio is created only after a user gesture, so `ensureAudio()` must be called from input handlers.
5. **HUD and effects**: hearts/bombs rendering, hit/bomb flashes, combo pop-ups, stage banners, screen shake (`screenShake`).
6. **Input**: keyboard (`keys` map, arrows/WASD, Space fire, X/Shift bomb, P/Esc pause), mouse (move to steer, click to fire via canvas coordinates), and touch (a left-side virtual joystick plus FIRE and BOMB buttons).
7. **Game state**: globals `score`, `lives`, `bombCount`, `stage`, `stageKills`, `chain`/`multiplier`, `player`, and the entity arrays `bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`, and `boss`. `resetGame()` reinitializes all of it.
8. **Spawning and combat**: `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp` (drop odds: 6% life, 8% bomb, 20% weapon), `fireWeapon` (Vulcan spread or Laser lanes, level 1–3), `applyPowerUp`, and the bomb (`useBomb`, which clears enemies and damages the boss).
9. **Boss AI**: `BOSS_PATTERNS` is an array of bullet-pattern functions (radial, aimed spread, columns, spiral, cross, homing, double ring) that `updateBoss` cycles through with telegraphs. `killBoss` advances the stage.
10. **Flow**: `loseLife`, `endGame`, `togglePause`.
11. **Render and loop**: `draw()` paints everything onto the canvas. `loop(now)` computes `dt` (clamped to 0.05 s), calls `trackPerf`, then `update(dt)` and `draw()` while the game is running and not paused. It is driven by `requestAnimationFrame`.

**Stage progression:** each stage spawns regular waves until `stageKills >= KILLS_PER_STAGE`, then `spawnBoss()` runs once (`bossWarned` guards it). Killing the boss resets `stageKills` and moves to the next stage. Boss HP is `(70 + stage * 35) * difficulty.bossHpMul * design.hpMul`.

**Difficulty** (`DIFFICULTIES`) sets starting lives, starting bombs, spawn intervals, enemy speed, enemy fire chance, and boss HP multiplier. It is chosen on the start overlay and persisted.

## Conventions

- **Keep it one file.** The project is deliberately a single self-contained HTML file with no external dependencies. Do not add npm packages, a build step, or external asset files unless the user asks for that change.
- **Keep the closure.** New top-level logic belongs inside the existing IIFE, not on `window`.
- **Frame-rate-independent motion.** Movement and timers multiply by `dt` (seconds), which is clamped in `loop`. Preserve this when adding entities.
- **Respect the two effect levers.** `reducedMotion` (user toggle, defaults to the OS `prefers-reduced-motion` setting) gates screen shake, flashes, and pulses. `quality` (automatic, from frame timing) reduces shadows, glows, and particle counts. New visual effects should check the appropriate one.
- **Accessibility.** Announce significant state changes through the `announce()` helper, which writes to the `srLive` live region.
- **Mobile first.** Layout is designed to work at phone widths with touch controls. Test touch changes at small viewports, not only with the mouse or keyboard.
- **Match existing style.** Two-space indentation, single quotes in JS, short `const` names for tuning tables, and comments as `// ---------- Section ----------` banners.

## Testing

There is no automated test suite. Verify changes by:

1. Opening `index.html` in a browser and playing through at least one stage boss on each difficulty you touched.
2. Checking the browser console for errors (the last commit notes a headless Playwright run was used to check firing, collisions, pickups, boss patterns, and stage advance with no console errors).
3. Checking touch controls with a mobile-sized viewport when the input code or layout changes.

If you add tests, do not add a framework dependency without asking first.

## Git

- Development branch for this session: `claude/gifted-tesla-e0p9f0`. Commit and push there (`git push -u origin <branch>`); do not push to other branches without permission.
- Do not open a pull request unless explicitly asked.
