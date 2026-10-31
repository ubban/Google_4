# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo is **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs in the browser on a single HTML5 canvas. There is no build step, no bundler, no package manager, and no test suite.

- `index.html` — the entire game: markup (~lines 1–360), CSS (`<style>`, ~lines 10–360), and one inline `<script>` IIFE (~lines 422–1667) containing all game logic, rendering, audio, and input.
- `README.md` — a stub; it contains no setup instructions.

To run the game, open `index.html` in a browser (or serve the directory with any static server). Nothing needs to be installed.

## Repository layout and conventions

- Keep the game as a single self-contained file unless the user asks for a split. Do not add a build system, framework, or dependencies without an explicit request.
- Everything lives inside one `(function () { 'use strict'; ... })();` IIFE. Do not leak globals; new helpers should be `function` declarations or `const`s inside that IIFE.
- Style is `const` by default, `let` for mutable state, and small named functions. Match the surrounding formatting (2-space indent, single quotes in JS, `//` section banners like `// ---------- Canvas sizing ----------`).
- Game tuning lives in data tables near the top of the script. Prefer editing those tables over hard-coding numbers in logic:
  - `DIFFICULTIES` — spawn timing, enemy speed, enemy fire chance, starting lives/bombs, boss HP multiplier.
  - `KILLS_PER_STAGE` — kills required before a boss warning.
  - `THEMES` — background palettes; cycled by `(stage - 1) % THEMES.length`.
  - `ENEMY_TYPES` — `grunt`, `interceptor` (weaving), `tank` (4 HP, stage 2+).
  - `BOSSES` — six bosses, cycled per stage; each has a name, palette, and `hpMul`.
  - `BOSS_PATTERNS` — boss bullet patterns (radial, aimed spread, columns, spiral, cross, homing, double ring).
- Canvas logical size is fixed at `BASE_W × BASE_H` = 480 × 720 and scaled to fit the window (max scale 1.6). Do not change the logical size without checking `resize()`, the touch joystick, and `player` start position.

## Architecture of the game loop

- `loop(now)` is driven by `requestAnimationFrame`. `dt` is clamped to 0.05 s. Each frame it calls `trackPerf(dt)`, then — only when `running && !paused` — `update(dt)` followed by `draw()`.
- `update(dt)` handles player movement (keyboard, mouse-follow, touch joystick), firing (`fireWeapon`), enemy spawning, enemy and bullet movement, collisions, power-ups, and boss logic (`updateBoss`).
- `draw()` renders everything to the canvas. Parallax clouds come from `drawClouds`; planes and bosses from `drawPlane` and `drawBossShip`.
- **Adaptive quality:** `trackPerf` keeps a 45-frame rolling average and steps a `quality` level (0 full, 1 medium, 2 low) up or down. Rendering code should respect `quality` when adding expensive effects.
- **Reduced motion:** the `reducedMotion` flag defaults from `prefers-reduced-motion` and can be toggled in the UI. When it is on, `screenShake` is scaled down and capped in duration, and the hit/bomb flash effects are skipped. New visual effects should respect this flag.
- **Stage flow:** after `KILLS_PER_STAGE` kills, `bossWarned` is set and a boss spawns 1.2 s later (`spawnBoss`). Killing the boss (`killBoss`) adds 500 points, increments `stage`, and resets `stageKills`.
- **Lives and game over:** `loseLife()` grants 1.5 s of invulnerability and resets the combo chain. `endGame()` saves the best score to `localStorage` if beaten and shows the overlay.

## Persistent state

Only four keys are stored, all in `localStorage` and all prefixed `skyfighter.`:

| Key | Meaning |
| --- | --- |
| `skyfighter.bestScore` | Highest score achieved |
| `skyfighter.muted` | `'1'` when sound is muted |
| `skyfighter.difficulty` | `easy` / `normal` / `hard` |
| `skyfighter.reducedMotion` | `'1'` / `'0'` override of the OS reduced-motion preference |

Reads and writes to `localStorage` are not wrapped in try/catch. If you add storage, wrap it so the game still runs when storage is blocked (private mode, disabled site data).

## Audio

Audio is synthesized with the Web Audio API — there are no audio files. `ensureAudio()` creates the context lazily on the first user gesture (required by browsers). Sound effects are exposed through the `sfx` object (`beep`, `noiseBurst`, and named effects such as `powerup`, `hit`, `bossDown`, `stageClear`, `gameover`). Add new sounds to `sfx` rather than calling Web Audio directly from gameplay code.

## Controls (keep these in sync with the on-screen hint)

- **Desktop:** mouse to steer and click to fire; Arrow keys or WASD to move; Space to fire; X or Shift to drop a bomb; P or Esc to pause.
- **Touch:** drag on the left zone to fly with a virtual joystick; tap FIRE; tap BOMB.

The on-screen hint is in `#controlsHint` (`index.html`, ~line 417). If you change a binding, update that text and the `keydown` handler (~line 769) together.

## Accessibility

- A polite live region (`#srLive`) is updated through `announce()` for lives, stage changes, and game over. Use `announce()` for any new important game-state change.
- Interactive controls are real `<button>`s with labels; keep it that way when adding UI.
- Difficulty buttons use `.diffBtn` with `data-diff`. Clicks are ignored while a run is active; they take effect between runs or while paused.

## Working in this repo

- **No automated tests or linter exist.** Verify changes by loading `index.html` in a browser and playing through the affected path. For logic changes, consider a quick headless check (for example, with the pre-installed Playwright/Chromium) that loads the page, starts a game, and checks for console errors.
- Before committing, run a basic syntax check on the inline script if you have Node available (for example, extract the `<script>` body and run `node --check`).
- Keep commits focused. Commit messages should describe the gameplay or code change, not the tool used to make it.
- Do not add new top-level files (such as `package.json`, build configs, or assets) unless the user asks for them.
- When a change affects balance (spawn rates, HP, boss patterns), say so in the commit message so it can be tuned deliberately.
