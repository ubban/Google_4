# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo is a small, dependency-free browser game: **Sky Fighter**, a Raiden-style vertical shoot-'em-up. The entire game lives in one file, `index.html`, which contains inline CSS (`<style>`, roughly lines 10–362), the canvas and HUD markup (roughly lines 363–421), and one inline IIFE script (`<script>`, starting around line 422).

The repository is otherwise minimal:

- `index.html` — the whole game (~1,670 lines).
- `README.md` — a stub containing only the repo name and a timestamp. It has no build or run instructions.
- `.git/` — history: an initial commit, then "Add Sky Fighter" on top.

## Running and testing

- There is no build step, package manager, bundler, linter, or test suite. Do not add one unless asked.
- To run the game, open `index.html` directly in a browser, or serve the directory with any static file server (for example `python3 -m http.server`).
- For UI or gameplay changes, verify in a real browser (a Playwright Chromium is often available in the cloud environment). Check that the title overlay appears, that Start begins a run, and that the HUD updates. Test at phone width as well as desktop width, since the game has touch controls.
- Do not introduce external runtime dependencies or CDN scripts. The game is intentionally self-contained and works offline.

## Architecture of `index.html`

Everything runs inside one `(function () { 'use strict'; ... })()` closure. Nothing is exported to `window`, so keep new code inside that closure.

Major sections, in the order they appear in the script:

1. **DOM references** — `const` lookups by element id (`gameCanvas`, `overlay`, `hearts`, `bombs`, `bossBarWrap`, `muteBtn`, `motionBtn`, `pauseBtn`, touch controls, and so on). If you add a HUD element, add its id here and in the markup.
2. **Persistence keys** — `STORAGE_BEST`, `STORAGE_MUTE`, `STORAGE_DIFF`, `STORAGE_REDUCED`, all prefixed `skyfighter.`. The best score, mute, difficulty, and reduced-motion settings are stored in `localStorage`.
3. **Adaptive performance** — `trackPerf(dt)` keeps a rolling window of 45 frame times and adjusts `quality` (0 full, 1 medium, 2 low) to lower the effects load when frame times are high.
4. **Tuning tables** — these are the main knobs for balance:
   - `DIFFICULTIES` (easy / normal / hard): spawn rates, enemy speed, enemy shoot chance, starting `lives`, starting `bombs`, and `bossHpMul`.
   - `KILLS_PER_STAGE` (18): kills needed to advance a stage.
   - `THEMES`: six sky palettes that cycle with stage.
   - `ENEMY_TYPES`: `grunt`, `interceptor` (weaving), and `tank` (multi-hit). Each has size, hp, speed multiplier, colors, score, and `minStage`.
   - `BOSSES`: six bosses (VANGUARD, BEHEMOTH, SPECTER, DREADNOUGHT, WRAITH, TITAN) with colors and `hpMul`.
   - `BOSS_PATTERNS`: the bullet patterns bosses use (radial, aimed spread, columns, spiral, cross, homing, double ring).
5. **Audio** — `ensureAudio()` lazily creates the Web Audio context, which browsers require to start after a user gesture. `beep`, `noiseBurst`, and the `sfx` object synthesize all sound effects. There are no audio files.
6. **Rendering helpers** — `drawPlane`, `drawBossShip`, `drawClouds`, and the main `draw()`. Drawing is done with 2D canvas primitives. There are no sprite assets.
7. **Input** — the `keys` map (keyboard, updated on `keydown`/`keyup`), touch joystick and fire/bomb buttons, and mouse steering and firing on the canvas.
8. **Game state and logic** — `resetGame()`, `spawnEnemy()`, `spawnBoss()`, `maybeSpawnPowerUp()`, `fireWeapon()`, `useBomb()`, `update(dt)`, `applyPowerUp()`, `updateBoss(dt)`, `killBoss()`, `loseLife()`, `endGame()`, `togglePause()`.
9. **Main loop** — `loop(now)` runs `update` and `draw` each animation frame. The Start button in the overlay calls `resetGame()` and hides the overlay. The script ends by calling `requestAnimationFrame(loop)`.

### Conventions

- Use `dt`-scaled movement in `update`, not per-frame constants, so the game runs at the same speed at different frame rates.
- Use the `announce(text)` helper for screen-reader updates, and keep the `aria-live` region current when state changes meaningfully.
- Respect `reducedMotion`. Screen shake, flashes, and the vignette pulse should be suppressed or softened when it is set. Check the `reduced-motion` body class.
- Keep the visual language consistent: the HUD is pill-style, the overlay is a glassmorphic card, and the sky palette comes from `THEMES`. Don't hard-code new colors that clash with the current theme.
- Sound effects must go through the existing `sfx`/`beep`/`noiseBurst` helpers and honor `muted`.
- Keep the game responsive. The base logical resolution is 480×720 (`BASE_W`, `BASE_H`), and `resize()` scales it to fit the viewport. Preserve phone-width layout and the touch controls.

## Persistence and state

- Only `localStorage` is used, and only for the four `skyfighter.*` keys. Wrap new storage access in a `try/catch` in case storage is blocked (for example in private browsing).
- Best score updates at game end (`endGame`, line ~1477). Don't overwrite it on pause or restart.

## Gameplay rules (for reference when changing balance)

- Controls: mouse move steers, click fires; arrows or WASD move; Space fires; X or Shift drops a bomb; P or Esc pauses.
- Lives and bombs come from the selected difficulty. Power-ups can raise weapon level (Vulcan / Laser, 3 levels), grant a bomb, or grant an extra life.
- Bombs (`useBomb`) clear enemy bullets and regular enemies and deal 24 damage to the boss, if one is present.
- Kills build a chain that multiplies score. Taking a hit (`loseLife`) and bosses being defeated both break the chain.
- After `KILLS_PER_STAGE` (18) kills in a stage, a boss spawns 1.2 seconds later, with a health bar. Defeating the boss (`killBoss`) advances `stage`, resets `stageKills`, and the theme cycles with `(stage - 1) % THEMES.length`.
- Losing a life grants 1.5 seconds of invulnerability. Losing the last life ends the run.

## Working guidelines

- Keep changes inside `index.html` unless there is a clear reason to split it. Splitting it would mean adding a build or module story, which the project does not have today.
- Make the smallest change that solves the problem. Don't reformat unrelated code, and don't rename the tuning constants without a reason.
- Balance changes belong in the tuning tables, not scattered through logic. If you change a number, say in the commit message what changed and why.
- The project uses plain ES2017+ syntax and `const`/`let`, with no transpilation. Match the surrounding style (two-space indentation, single quotes in JS, terse helper names).
- After editing, re-check that the file still parses (for example by loading it in a headless browser and watching for console errors) before committing.

## Git

- Development branch: `claude/gifted-tesla-xxrdeb`. Commit and push there. Don't push to other branches without permission.
- Don't open a pull request unless explicitly asked.
