# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Sky Fighter is a Raiden-style vertical shoot-'em-up that runs in the browser. The entire game lives in one file, `index.html`: markup, CSS, and JavaScript. There is no build step, no bundler, no package manager, and no runtime dependencies.

`README.md` is a stub and does not describe the game. Treat `index.html` as the source of truth.

## Repository layout

```
index.html   # The whole game: <style>, DOM overlay/HUD, and one <script> IIFE
README.md    # Minimal project stub
```

## Running and verifying

- Run the game by opening `index.html` directly in a browser. No server is required.
- There is no test suite, linter, formatter, or CI configured. Do not invent commands for them.
- To verify a change, load the page in a browser and play through the affected path. For changes to gameplay, check the start overlay, a few stages, boss fights, game over, and the high score. For HUD or layout changes, check both desktop and a narrow/touch viewport.
- For a quick syntax check of the inline script, you can extract it and run `node --check` on it. Do not commit the extracted file.

## Code architecture

All game code sits inside a single `(function () { 'use strict'; ... })();` IIFE in the `<script>` block at the bottom of `index.html`. Nothing is exposed on `window`. Keep it that way unless there is a clear reason to change it.

Rough order of sections inside the IIFE, in the order they appear:

1. **DOM references**: `const xEl = document.getElementById(...)`. All element lookups happen once, at the top.
2. **Persistence keys**: `STORAGE_*` constants for `localStorage`, all prefixed `skyfighter.`.
3. **Perf tracking**: `trackPerf` samples frame times and steps a `quality` level (0 full, 1 medium, 2 low) to keep the frame rate up. Several draw paths branch on `quality`, for example shadow blur and particle counts.
4. **Tuning tables**: `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES` (per-stage sky palettes), `ENEMY_TYPES`, `BOSSES`. Balance changes should usually be made here, not in the update logic.
5. **Audio**: a lazily created `AudioContext` (`ensureAudio`) and small synth helpers (`beep`, `noiseBurst`, the `sfx` object). Nothing is loaded from files.
6. **HUD and effects**: `renderHearts`, `renderBombs`, `flashHit`, `showComboPop`, `showStageBanner`, `screenShake`, all of which respect `reducedMotion`.
7. **Input**: keyboard (`keys` map keyed by `e.code`), mouse on the canvas, and touch controls (a left-side joystick plus FIRE and BOMB buttons). Touch UI is shown only when `isTouchDevice` is true.
8. **Game state and logic**: `resetGame`, `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`, `addScore`/combo chain, `useBomb`, `fireWeapon`, `update(dt)`, `applyPowerUp`, the boss attack patterns (`BOSS_PATTERNS` and `fire*` helpers), `updateBoss`, `killBoss`, `loseLife`, `endGame`, `togglePause`.
9. **Rendering**: `drawPlane`, `drawBossShip`, `drawClouds`, and `draw()`. The canvas is 480×720 in logical pixels (`BASE_W`, `BASE_H`) and is scaled by CSS in `resize()`.
10. **Main loop**: `loop(now)` clamps `dt` to 50 ms, calls `update` and `draw` only when `running && !paused`, then requests the next frame.

### Conventions

- **Time**: all movement and timers are `dt`-based (seconds). Do not tie gameplay to frame counts, except where a `frame` counter is already used for deterministic patterns.
- **State**: game state is module-level `let` variables inside the IIFE. Reset them in `resetGame()`. If you add state, add its reset there too.
- **Entities**: enemies, bullets, and power-ups are plain objects in arrays, removed when they go off-screen or die. Collision uses `rectsOverlap`.
- **Persistence**: only use `localStorage` through the existing `STORAGE_*` keys, and keep the `skyfighter.` prefix. Wrap new reads with sensible defaults, as the existing code does.
- **Accessibility**: announcements go through `announce()` into the `#srLive` region. Reduced motion (`reducedMotion`, with a system preference default) must suppress shake, flash, and similar effects.
- **Style**: 2-space indentation, single quotes, semicolons, `const` by default, arrow functions for callbacks, and `addEventListener` for all wiring. Match the existing comment style. Section headers use `// ---------- Name ----------`.
- **No dependencies**: do not add external scripts, fonts, or libraries. The page is self-contained.

## Controls (for reference when testing)

- Keyboard: arrows or WASD to move, Space to fire, X or Shift to bomb, P or Esc to pause.
- Mouse: move to steer, click to fire.
- Touch: drag on the left side to fly, FIRE and BOMB buttons on the right.

## Gameplay notes

- Difficulty (Easy / Normal / Hard) is picked on the start overlay and stored in `skyfighter.difficulty`. It sets spawn rate, enemy speed, enemy fire chance, starting lives, starting bombs, and boss HP multiplier.
- Each stage ends after `KILLS_PER_STAGE` kills, which triggers a boss fight. Defeating the boss advances the stage and cycles through `BOSSES` and `THEMES`.
- Weapon state is `player.weapon` (`'vulcan'` or `'laser'`) plus `player.level` (1–3). Each weapon power-up raises the level; past level 3 it switches weapon type and resets the level to 1. `weaponLabel()` feeds the HUD.
- Enemy types unlock by stage through `minStage` in `ENEMY_TYPES`.
- Best score is stored in `skyfighter.bestScore`.

## Git workflow

- Development branch for this work: `claude/gifted-tesla-uwq45u`. Do not push to `main` without explicit permission.
- Keep commits small and describe the gameplay or UI change in the subject line.
- Do not open a pull request unless one is requested.
