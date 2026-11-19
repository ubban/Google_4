# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo is **Sky Fighter**, a single-file, Raiden-style vertical shoot-em-up played in the browser. There is no build step, no package manager, no test suite, and no dependencies.

Files:

- `index.html` (~1,670 lines) — the entire game: markup, CSS (`<style>`, roughly lines 10–360), canvas element, and all JavaScript (one IIFE `<script>` starting around line 422).
- `README.md` — a stub. Its second line is a timestamp, not documentation.

To play, open `index.html` in a browser. There is nothing to install or compile.

## Repository conventions

- Keep the game self-contained in `index.html`. Do not introduce a bundler, framework, or external asset dependency unless the user asks for it.
- Keep all JS inside the existing IIFE and `'use strict'` block. Do not add globals.
- Use the existing naming style: `camelCase` for functions and variables, `SCREAMING_SNAKE_CASE` for constants and `STORAGE_*` keys.
- Persisted settings use `localStorage` keys prefixed with `skyfighter.` (`bestScore`, `muted`, `difficulty`, `reducedMotion`). Wrap new storage access so the game still works if storage is unavailable, and namespace any new key the same way.
- Gameplay tuning lives in data tables near the top of the script, not inline in logic:
  - `DIFFICULTIES` — spawn rates, enemy speed and fire chance, lives, bombs, boss HP multiplier.
  - `ENEMY_TYPES` — `grunt`, `interceptor`, `tank`, with size, HP, speed, score, and `minStage`.
  - `BOSSES` — boss palette, name, and HP multiplier. `BOSS_PATTERNS` (around line 1372) holds the attack patterns.
  - `THEMES` — one sky/cloud/sun palette per stage; the theme cycles as stages advance.
  - `KILLS_PER_STAGE` — kills needed to advance a stage.
  Prefer changing these tables over hard-coding new numbers.
- The logical playfield is `BASE_W = 480` × `BASE_H = 720`, scaled to the viewport by `resize()`. Coordinates in game logic should use these base units.

## Architecture

The game loop is a single `requestAnimationFrame` loop (`loop`, near the end of the script):

1. Compute `dt` in seconds, clamped to 0.05 so tab switches do not teleport entities.
2. `trackPerf(dt)` samples frame times and adjusts a `quality` level (0 full, 1 medium, 2 low) to shed effects on slow devices.
3. If the game is running and not paused: `update(dt)`, then `draw()`.

Main sections, in file order:

- **Audio** — `ensureAudio`, `beep`, `noiseBurst`, and the `EXPLODE_SFX` mapping. Audio is created lazily on the first user gesture; keep it that way for autoplay policy.
- **HUD / feedback** — `renderHearts`, `renderBombs`, `flashHit`, `flashBomb`, `showComboPop`, `showStageBanner`, `screenShake`, `announce` (screen-reader live region).
- **Input** — keyboard (`keys[e.code]`), touch joystick and fire/bomb buttons, and mouse. Pause is on `P`/`Escape`; bomb is on `X` or either `Shift`.
- **Game state** — `resetGame`, `update`, `loseLife`, `endGame`, `togglePause`.
- **Spawning and combat** — `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`, `applyPowerUp`, `fireWeapon`, `useBomb`, `addScore`, `breakChain`, `killBoss`.
- **Boss attacks** — `fireRadial`, `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`, `fireCross`, `fireHomingSeeded`, `fireDoubleRing`, driven by `updateBoss`.
- **Rendering** — `draw`, `drawPlane`, `drawBossShip`, `drawClouds`. All drawing is done with the 2D canvas context; there is no WebGL.

Game-state flags worth knowing before editing: `running`, `paused`, `firing`, `keys`, `quality`, `reducedMotion`, `muted`, `difficulty`, `bestScore`.

## Accessibility and settings

- Respect `prefers-reduced-motion` on first load, and let the user override it with the motion toggle. When reduced motion is on, shake and flash effects are suppressed. Any new screen-shake, flash, or particle effect must check `reducedMotion`.
- Announce meaningful state changes (stage advance, game over, pause, etc.) through `announce()` so screen readers get them.
- Touch controls must keep working: use `touchstart`/`touchend` with `preventDefault` on the on-screen buttons, and don't break the joystick when adding UI.

## Working on this repo

- **Run it:** open `index.html` directly, or serve the folder with any static server (e.g. `python3 -m http.server`) and visit it in a browser. There are no automated tests, so verify changes by playing them in a browser, including at phone width and with touch input where relevant.
- **Syntax check after JS edits:** because there is no build, a quick sanity check is to extract the `<script>` block and run `node --check` on it (for example, write it to a scratch file outside the repo first).
- **Scope:** keep changes minimal and focused. Do not reformat unrelated code or reorder the data tables.
- **Git:** the working branch for this environment is `claude/gifted-tesla-dywue4`. Commit and push only when asked, and do not open a pull request unless the user explicitly requests one.

## Things to avoid

- Don't split `index.html` into multiple files or add a `package.json` without an explicit request.
- Don't store secrets, tokens, or personal data in the repo or in `localStorage`.
- Don't remove the `'use strict'` block or the IIFE wrapper.
- Don't hard-code tunable numbers in the update/draw logic when they belong in the existing data tables.
