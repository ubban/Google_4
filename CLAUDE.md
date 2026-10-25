# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up that runs in the browser. The entire game is one self-contained file, `index.html` (~1,670 lines): inline CSS in `<style>`, markup in `<body>`, and one inline `<script>` IIFE. There is no framework, no bundler, no `package.json`, and no test suite. `README.md` is a placeholder.

## Running it

No build step. Open `index.html` directly in a browser, or serve the directory with any static server (for example `python3 -m http.server`) and load it from `localhost`.

## Repository layout

| Path | Purpose |
| --- | --- |
| `index.html` | The whole game: layout, styles, and logic |
| `README.md` | Placeholder |
| `CLAUDE.md` | This file |

## Architecture of `index.html`

Layout, top to bottom:

1. `<head>`: viewport meta (no user scaling), theme color, and an inline SVG favicon.
2. `<style>`: all CSS. Overlays, HUD, touch controls, and the boss health bar are styled here.
3. `<body>`: a `<canvas id="gameCanvas">` (logical size `BASE_W × BASE_H` = 480×720), a HUD (score, best, multiplier, weapon, hearts, bombs, mute/motion/pause buttons), touch controls (`#moveZone` joystick, `#fireBtn`, `#bombBtn`), the start/game-over overlay, and a `#srLive` live region for screen readers.
4. `<script>`: one IIFE with `'use strict'`. Everything is private to the closure, so there are no globals.

Inside the IIFE, code is grouped by `// ----------` section comments and by function. Key parts:

- **Element refs**: every DOM node is fetched once at the top with `getElementById` or `querySelectorAll`.
- **Persistence**: `localStorage` keys are prefixed `skyfighter.` (`bestScore`, `muted`, `difficulty`, `reducedMotion`). Every read is wrapped in a way that defaults cleanly when unset.
- **Adaptive quality**: `trackPerf(dt)` averages the last 45 frame times and steps a `quality` level (0 full, 1 medium, 2 low) up or down to hold roughly 40–56 FPS. Use `quality` to scale effects; do not hard-code effect density.
- **Config tables** (edit these to tune the game, not the logic):
  - `DIFFICULTIES`: spawn timing, enemy speed, shoot chance, starting lives and bombs, and a boss HP multiplier for `easy`, `normal`, and `hard`.
  - `KILLS_PER_STAGE` (18): kills needed to trigger a boss.
  - `THEMES`: six sky palettes that cycle with each stage.
  - `ENEMY_TYPES`: `grunt`, `interceptor` (weaves), and `tank` (4 HP). Each has size, HP, speed, colors, score, and the minimum stage it appears in.
  - `BOSSES`: six bosses (VANGUARD, BEHEMOTH, SPECTER, DREADNOUGHT, WRAITH, TITAN) with colors and an HP multiplier. `BOSS_PATTERNS` is indexed by boss, so each boss has its own bullet pattern set.
- **Audio**: a lazily created `AudioContext` (`ensureAudio()`), with `beep` and `noiseBurst` primitives and a named `sfx` table. Audio starts only after a user gesture.
- **Input**: keyboard (`keys[e.code]`), mouse, and touch (virtual joystick and buttons). Arrows or WASD move the ship. Space fires. Mouse movement steers the ship, and clicking the canvas fires. Pause is `P` or `Esc`. Bomb is `X` or `Shift`. Touch uses the left-side joystick plus the FIRE and BOMB buttons.
- **Game state**: top-level `let` variables (`score`, `lives`, `stage`, `chain`, `multiplier`, `bombCount`, and so on) plus arrays for `bullets`, `enemies`, `enemyBullets`, `powerUps`, and particles. `resetGame()` restores the defaults.
- **Loop**: `loop(now)` computes `dt` (clamped to 50 ms), calls `trackPerf`, then runs `update(dt)` and `draw()` only while `running && !paused`. `requestAnimationFrame` drives it.
- **Update vs. draw**: `update` moves entities, handles collisions, and advances stages and bosses. `draw` only renders. Keep game logic out of `draw`.
- **Power-ups**: `maybeSpawnPowerUp` drops `life`, `bomb`, or `weapon` pickups. Weapon pickups raise the level (1–3) and then toggle between `vulcan` and `laser`.

## Conventions

- **Style**: two-space indentation, single quotes in JS, semicolons, `const` by default and `let` only when reassigned. Match the existing terse style and its section-divider comments (`// ---------- Name ----------`).
- **Keep it one file.** Do not split out modules, add a build step, or add dependencies unless the user asks. Everything should keep working by opening `index.html` directly from disk.
- **Scope**: all new code goes inside the existing IIFE. Do not add globals.
- **Tuning**: change balance values in the config tables at the top of the script, not inside `update` or `draw`.
- **Accessibility and mobile**: respect `reducedMotion` for shake, flash, and pulse effects. Keep the layout usable at phone widths with touch controls, and keep `#srLive` announcements for important state changes (use `announce()`).
- **Performance**: avoid per-frame allocations in hot paths where practical. Prefer the existing array-splice-in-reverse pattern when removing entities during iteration.

## Testing and verification

There is no automated test suite or linter. Verify changes by:

1. Opening `index.html` in a browser (or a local static server) and checking the console for errors.
2. Playing through the affected path: start, fire, bomb, pause and resume, the stage 1 boss, game over, and the best-score update.
3. Checking the difficulty buttons, mute, and reduced-motion toggles, since they persist to `localStorage`.
4. Checking the touch layout at a phone-sized viewport. Chrome DevTools device mode is enough.

If you add a test harness or linter, document the commands here.

## Git workflow

- Develop on the branch the session specifies. Do not push to `main` directly.
- Use clear, descriptive commit messages in the imperative mood.
- Do not open a pull request unless the user asks for one.
