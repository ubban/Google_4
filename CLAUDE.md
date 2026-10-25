# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Sky Fighter is a Raiden-style vertical shoot-'em-up that runs in the browser. The entire game lives in one file, `index.html`, with inline CSS and JavaScript. `README.md` is a placeholder and has no setup steps.

There is no build step, no package manager, no dependencies, and no test suite. To run the game, open `index.html` in a browser.

## Repository layout

- `index.html` — the whole game: markup (`<body>`), styles (`<style>`, lines 10–359), and the game script (`<script>`, lines 422–1668).
- `README.md` — placeholder.

## Architecture of `index.html`

- **Single IIFE.** All game code sits inside `(function () { 'use strict'; ... })();`. Nothing is exposed on `window`. Keep new code inside that scope.
- **Fixed logical resolution.** The canvas is `480 x 720` (`BASE_W`, `BASE_H`). `resize()` scales the displayed size to fit the window, capped at 1.6x. Game coordinates always use the logical size.
- **Main loop.** A single `requestAnimationFrame` loop (`loop`) computes `dt` (capped at 0.05 s), calls `trackPerf`, then `update(dt)` and `draw()` while the game is running and not paused.
- **Adaptive quality.** `trackPerf` watches a 45-frame average and steps `quality` between 0 (full), 1 (medium), and 2 (low) when frame time drifts above about 40 FPS or below about 56 FPS.
- **Difficulty table.** `DIFFICULTIES` holds `easy`, `normal`, and `hard` settings (spawn rates, enemy shoot chance, starting lives and bombs, boss HP multiplier). Select with `setDifficulty`.
- **Content tables.** `THEMES` (background palettes) and `BOSSES` (six boss designs) are indexed by `(stage - 1) % length`, so they repeat after six stages. `ENEMY_TYPES` (`grunt`, `interceptor`, `tank`) is keyed by name with a `minStage` gate. These are plain data; add new content there first.
- **Progression.** `KILLS_PER_STAGE` (18) kills trigger a boss warning and then `spawnBoss()` after about 1.2 s. `killBoss()` advances `stage`.
- **Weapons and power-ups.** `applyPowerUp` handles `life`, `bomb`, and `weapon` pickups. Weapons are `vulcan` and `laser`, each with up to three levels. Boss attack patterns live in `BOSS_PATTERNS`, built from the `fire*` helpers.
- **Scoring.** `addScore` applies a chain multiplier (`1 + min(9, floor(chain / 5))`). The chain resets after `chainTimer` runs out (`breakChain`).
- **Audio.** Web Audio is created lazily by `ensureAudio()`, which runs on the first user gesture. Sound effects are in the `sfx` object. `beep` and `noiseBurst` are the primitives, and `panFor` gives stereo positioning. Respect the `muted` flag.
- **Accessibility.** `announce()` writes to the `#srLive` live region. Reduced motion (`reducedMotion`) turns off screen shake, flash, and pulse effects. It defaults to the OS `prefers-reduced-motion` setting until the user toggles it.
- **Persistence.** `localStorage` keys are `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, and `skyfighter.reducedMotion`. Keep the `skyfighter.` prefix for new keys. All reads currently assume storage is available.

## Controls

- Desktop: mouse to steer and click to fire, or arrows/WASD to move, Space to fire, X or Shift for a bomb, P or Esc to pause.
- Touch: a virtual joystick on the left (`moveZone`), with FIRE and BOMB buttons on the right (`actionZone`). Touch controls are enabled when `isTouchDevice` is true.

## Working conventions

- Keep everything in `index.html` unless the user asks for a split. Do not introduce a build step or external dependencies without asking.
- Match the existing style: two-space indentation, `const` for constants, `UPPER_SNAKE_CASE` for config tables and storage keys, and short section comments in the form `// ---------- Name ----------`.
- Use `addEventListener` for handlers, following the current pattern.
- Keep DOM lookups at the top of the IIFE, as the file does.
- When changing gameplay numbers, prefer editing the data tables (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `THEMES`) over hard-coding values in logic.

## Verifying changes

There are no automated tests or linters. Verify changes by:

1. Opening `index.html` in a browser and checking the browser console for errors.
2. Playing through at least one stage: shooting, collecting power-ups, using a bomb, reaching a boss, and dying.
3. Checking the affected difficulty and mobile/touch paths if you changed input or layout.
4. Checking that pause, mute, reduced-motion, and best-score persistence still work after reloading the page.

If you cannot run a browser, say so explicitly rather than claiming the change works.
