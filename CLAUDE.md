# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo is a small, dependency-free browser game, **Sky Fighter** — a Raiden-style vertical shoot-'em-up. The entire game lives in a single file, `index.html` (~1,670 lines): inline `<style>`, the HTML/UI markup, and one inline `<script>` IIFE that drives an HTML5 `<canvas>` (2D context).

- `index.html` — the whole game (markup, CSS, JavaScript).
- `README.md` — a stub title; there is no other documentation.

There is no build step, no package manager, no bundler, no test suite, and no linter configured. Nothing needs to be installed.

## Running and verifying

- Open `index.html` directly in a browser (double-click or `file://`), or serve the directory with any static server, e.g. `python3 -m http.server` from the repo root, then visit `http://localhost:8000`.
- There are no automated tests. Verify changes by playing the game in a browser. For UI/gameplay changes, exercise the affected path (start, fire, bomb, pause, stage transition, boss, game over, difficulty change, mute, reduced-motion toggle) and check the browser console for errors.
- Test on a narrow/touch viewport as well when touching layout or input: the page has a mobile layout and on-screen touch controls.

## Code layout inside `index.html`

- **`<style>`** (around lines 10–360): all CSS for the HUD, overlay, touch controls, and the `.reduced-motion` class.
- **`<body>`** (around lines 361–421): the canvas (`#gameCanvas`), HUD elements, overlay (start / pause / game-over), touch controls, and a hidden `#srLive` live region for screen-reader announcements.
- **`<script>`** (starts around line 422): one IIFE with `'use strict'`. Sections, in rough order:
  - DOM references and `STORAGE_*` keys (`skyfighter.bestScore`, `.muted`, `.difficulty`, `.reducedMotion`).
  - Adaptive quality: `trackPerf` lowers/raises a `quality` level (0–2) based on average frame time.
  - Tuning data: `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES` (per-stage sky palettes), `ENEMY_TYPES`, `BOSSES`.
  - Audio: `ensureAudio`, `beep`, `noiseBurst`, and the `sfx` helpers (Web Audio, synthesized — no audio files).
  - HUD/FX helpers: hearts, bombs, combo pop, stage banner, screen shake, flashes.
  - Input: keyboard handlers, a virtual joystick and fire/bomb buttons for touch, and mouse/pointer steering via `canvasPointFromEvent`.
  - Game state: `resetGame`, `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`, `applyPowerUp`, `useBomb`, `fireWeapon`, `addScore`, `breakChain`.
  - Boss logic: `BOSS_PATTERNS` (bullet patterns such as radial, spiral, cross, homing) and `updateBoss` / `killBoss`.
  - Flow: `loseLife`, `endGame`, `togglePause`.
  - Rendering: `drawPlane`, `drawBossShip`, `drawClouds`, `draw`.
  - Main loop: `update(dt)` (logic) and `draw()` (rendering) driven by `loop(now)` via `requestAnimationFrame`. `dt` is clamped to 50 ms.

Line numbers are approximate and will drift; search for the function or constant name instead.

## Conventions

- **Keep it a single file.** Do not introduce a build system, framework, or external dependency unless the user explicitly asks. Styles and scripts stay inline.
- **Keep everything inside the IIFE.** New code should be added inside the existing `(function () { ... })();` so no new globals leak onto `window`. Only the inline handlers and DOM ids are used from outside.
- **Use `const`/`let`, `'use strict'` style, and the existing naming:** `UPPER_SNAKE` for tuning tables and storage keys, `camelCase` for functions and state, and `ctx`-based drawing.
- **Tune gameplay through the data tables** (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `THEMES`, `BOSS_PATTERNS`) rather than hard-coding numbers in logic.
- **Time-based movement:** update code multiplies by `dt` (seconds). Don't use per-frame increments that ignore frame rate.
- **Persistence:** `localStorage` is used for best score, mute, difficulty, and reduced-motion. Always use the existing `STORAGE_*` constants, and keep reads/writes tolerant of the storage being unavailable where practical.
- **Accessibility:** respect `prefers-reduced-motion` (the `reducedMotion` flag / `.reduced-motion` class) when adding screen-shake or flash effects. Announce important state changes through `announce()` to the `#srLive` region.
- **Audio** is synthesized with the Web Audio API. Audio must start only after a user gesture — `ensureAudio()` is called from input handlers; don't create an `AudioContext` on page load.
- **Input priority:** keyboard and joystick input take priority when active so the two never fight (see the comment near `canvasPointFromEvent`). Preserve this when changing controls.

## Controls (for reference when testing)

- Mouse: move to steer, click to fire.
- Arrows / WASD: move. Space: fire.
- X or Shift: bomb. P or Esc: pause.
- Touch: drag left side to fly, tap FIRE, tap BOMB.

## Git and workflow

- Default working branch for this session: `claude/gifted-tesla-ff4me9`.
- Commit messages should be short and descriptive of the change. Do not open a pull request unless explicitly asked.
- Before committing, load `index.html` in a browser and confirm there are no console errors on start and during play.

## Keeping this file current

Update this CLAUDE.md when the structure changes materially — for example, if the game is split into multiple files, a build or test step is added, or the controls or storage keys change. Keep it factual and short; prefer pointing to a function name over duplicating code.
