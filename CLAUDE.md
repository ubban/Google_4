# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` contains **Sky Fighter**, a Raiden-style vertical shoot-'em-up built as a single-page HTML5 canvas game. The entire game (markup, CSS, and JavaScript) lives in `index.html`. `README.md` is a placeholder and contains no build or run instructions.

There is no build system, package manifest (`package.json`), test suite, linter, or CI configuration. To run the game, open `index.html` in a browser, or serve the directory with any static file server.

## Repository layout

```
index.html   # The whole game: <style> (~lines 10-360), HTML overlay/HUD (~361-421), <script> (~422-end)
README.md    # Placeholder
```

## Architecture of `index.html`

The script is one IIFE (`(function () { 'use strict'; ... })();`) with no modules or external dependencies. Key sections, in source order:

- **DOM references**: every element is looked up once by id at the top of the script (`canvas`, `overlay`, `hud` elements, touch controls, boss bar, etc.). Keep new element lookups in this block.
- **Persistence keys**: `STORAGE_BEST`, `STORAGE_MUTE`, `STORAGE_DIFF`, `STORAGE_REDUCED` (`skyfighter.*` in `localStorage`). Reads and writes to `localStorage` are used for these settings and the best score.
- **Config tables**:
  - `DIFFICULTIES` (`easy` / `normal` / `hard`): spawn rate, enemy speed and fire chance, starting lives and bombs, boss HP multiplier.
  - `KILLS_PER_STAGE`: kills required to advance a stage.
  - `THEMES`: per-stage sky gradient, cloud and sun colors, and name.
  - `ENEMY_TYPES` (`grunt`, `interceptor`, `tank`): size, HP, speed, colors, score, and `minStage`.
  - `BOSSES`: six bosses, cycled by stage, each with hull/wing/core colors and an HP multiplier. Attack patterns are in `BOSS_PATTERNS` and the `fire*` helpers.
- **Audio**: a lazily created `AudioContext` (`ensureAudio`), with `beep`, `noiseBurst`, and a named `sfx` table. Audio is unlocked by a user gesture (`ensureAudio()` is called from start, fire, and bomb handlers).
- **Input**: keyboard (`keys[e.code]`), a touch joystick plus fire and bomb buttons, and mouse (`canvasPointFromEvent`, which converts client coordinates to the 480x720 logical space).
- **Layout**: the canvas uses a fixed logical resolution `BASE_W = 480`, `BASE_H = 720`, and `resize()` scales it to fit the viewport.
- **Game state and loop**: `resetGame()` initializes state. `update(dt)` advances the simulation, `draw()` renders, and `loop(now)` drives both via `requestAnimationFrame`. `dt` is clamped to 0.05s. Pause is handled by `togglePause()`.
- **Gameplay systems**: `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp` / `applyPowerUp`, `fireWeapon` (Vulcan and Laser, each with 3 levels), `useBomb` (clears the screen and damages bosses), the kill-chain multiplier (`addScore`, `breakChain`), `loseLife`, and `endGame`.
- **UI feedback**: `showComboPop`, `showStageBanner`, `screenShake`, `flashHit`, `flashBomb`, and `announce` (writes to the `srLive` aria-live region).
- **Performance tracking**: `trackPerf(dt)`.

## Conventions

- **Single file.** Keep the game in `index.html` unless the user asks for a split. Do not add dependencies, frameworks, or a build step without asking.
- **Vanilla JS, ES2015+ features only.** Match the existing style: `const`/`let`, arrow functions, no semicolon-less code, 2-space indentation, single quotes.
- **Gameplay tuning goes in the config tables** (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `THEMES`) rather than inline magic numbers where practical.
- **Respect reduced motion.** Visual effects such as screen shake and flashes should check the `reducedMotion` flag, as the existing code does.
- **Touch and desktop both matter.** New controls need both a keyboard/mouse path and a touch path, and the layout must work with `touch-action: none` and the safe-area insets already in the CSS.
- **Accessibility.** Announce important game events through `announce()` so they reach the `aria-live` region.

## Working in this repo

- There are no automated tests. Verify changes by loading `index.html` in a browser and playing through the affected path: start, fire, bomb, pause, a stage transition, a boss fight, game over, and the best-score persistence on reload. Check both desktop and a narrow mobile viewport.
- When changing rendering or input, also check the touch controls, because they share the same state variables as the keyboard path.
- Do not commit changes to `README.md` or other files unless the task calls for it.

## Git

- Develop on the branch specified by the task (currently `claude/gifted-tesla-h4jdst`). Do not push to other branches without permission.
- Write clear, descriptive commit messages in the imperative mood, matching the style of the existing history (for example, `Add Sky Fighter: ...`).
- Do not create pull requests unless explicitly asked.
