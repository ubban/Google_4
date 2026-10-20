# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo is a single-file browser game, **Sky Fighter** (a Raiden-style vertical shoot-'em-up).

- `index.html` is the entire game: markup (`<body>`), CSS (`<style>`, roughly lines 10–360), and one inline `<script>` IIFE (starting around line 422).
- `README.md` is a placeholder with no project description.
- There is no build step, package manager, test suite, linter, or CI configuration.

## Running the game

Open `index.html` directly in a browser, or serve the directory with any static server, for example:

```sh
python3 -m http.server 8000
```

Then visit `http://localhost:8000/`. Nothing needs to be installed.

## Code layout inside `index.html`

The script is wrapped in a single IIFE, so nothing leaks onto `window`. Within it, roughly in order:

1. **DOM lookups**: `const xxxEl = document.getElementById(...)` at the top of the script. Add new UI hooks here.
2. **Storage keys**: `STORAGE_*` constants (`skyfighter.*` prefix) for `localStorage`.
3. **Tuning tables**: `DIFFICULTIES`, `THEMES` (per-stage background palettes), `ENEMY_TYPES`, `BOSSES`, plus `KILLS_PER_STAGE`. Most balance changes belong here rather than in logic.
4. **Audio**: `ensureAudio()`, `beep()`, `noiseBurst()`, and the `sfx` object. Audio is built with the Web Audio API and created lazily on first user gesture.
5. **HUD helpers**: `renderHearts`, `renderBombs`, `showComboPop`, `showStageBanner`, `flashHit`, and similar.
6. **Canvas sizing**: `BASE_W × BASE_H` is 480×720. `resize()` scales the canvas CSS size to fit the window, capped at 1.6×. Game logic always works in base coordinates.
7. **Input**: keyboard (`keys[e.code]`), mouse-follow on the canvas (`mouseTarget`), and touch (virtual joystick, fire and bomb buttons).
8. **Game state and simulation**: `resetGame()`, `spawnEnemy()`, `spawnBoss()`, `fireWeapon()`, `useBomb()`, `update(dt)`, `addScore()`, and related helpers.
9. **Rendering**: `draw()`, `drawPlane()`, `drawBossShip()`, and particle/explosion code.
10. **Main loop**: `loop(now)` calls `trackPerf`, then `update` and `draw` when running and not paused. `dt` is clamped to 50 ms.

### Conventions to keep

- **Keep the single-file structure.** Don't split the game into modules or add a bundler unless the user asks for it.
- **Use `dt`-scaled movement.** Motion in `update()` multiplies by `dt`, so behavior is frame-rate independent. Follow that pattern for new movement.
- **Respect the quality level.** `quality` (0 = full, 1 = medium, 2 = low) is adjusted at runtime by `trackPerf()`. Effects that are expensive, such as `shadowBlur`, glows, and particle counts, should check `quality`, as the existing code does.
- **Respect reduced motion.** Screen shake and flashes are gated on the `reducedMotion` setting (initialized from `prefers-reduced-motion` and the saved preference). New visual effects that move the screen should honor it.
- **Keep accessibility hooks.** The `srLive` live region is used by `announce()` for screen-reader feedback. Use `announce()` for significant game events rather than writing to the DOM directly.
- **Persist only through the existing keys.** Best score, mute, difficulty, and reduced motion are stored in `localStorage` under `skyfighter.*`. Wrap any new storage access in a way that tolerates unavailable storage, since private browsing or blocked storage can throw.
- **Match the surrounding style.** The code uses `const`/`let`, arrow functions for callbacks, two-space indentation, single quotes in JS, and short section banners in the form `// ---------- Name ----------`. Keep new code consistent with that.

## Making changes

- Edit `index.html` directly. There is nothing to regenerate.
- Because there are no tests, verify changes by loading the page in a browser and playing through the affected path: start, fire, bomb, pause, game over, and restart. Check the console for errors. Also check the layout at a narrow (phone) viewport, since touch controls appear on touch devices.
- For balance tweaks, change the values in `DIFFICULTIES`, `ENEMY_TYPES`, or `BOSSES` and playtest all three difficulties.
- Keep changes focused. Don't reformat unrelated parts of `index.html`, which makes diffs hard to review.

## Git

- Development happens on feature branches (for example `claude/...`). Don't push to other branches without explicit permission.
- Write clear, descriptive commit messages that explain why the change was made.
