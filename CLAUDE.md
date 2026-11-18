# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo is **Sky Fighter**, a single-file, browser-based Raiden-style vertical shoot-'em-up. There is no build step, no package manager, and no test suite.

Files:

- `index.html` — the entire game: inline `<style>`, markup, and one inline `<script>` (~1,670 lines). Rendering is a 2D `<canvas id="gameCanvas">`; the HUD and overlays are DOM elements.
- `README.md` — stub (title only).

Git: the default branch is the one checked out at clone time; development for this session happens on `claude/gifted-tesla-j17q9b`. Remote is `origin` → `https://github.com/ubban/Google_4`.

## Running and testing

- Open `index.html` directly in a browser, or serve the folder with any static server (e.g. `python3 -m http.server`).
- There are no automated tests or linters configured. Verify changes by playing the game in a browser; for scripted checks, Playwright is available (see the environment notes), and `chromium.launch()` needs no extra options.
- Test both desktop (keyboard) and touch layouts when touching input or HUD code. Test at phone width (the canvas scales to fit; `#hud` wraps).

## Code structure (inside `index.html`'s script)

Roughly in file order:

- **Config constants**: `STORAGE_*` keys (localStorage: best score, mute, difficulty, reduced motion), `DIFFICULTIES` (easy / normal / hard: spawn rate, enemy speed, shot chance, lives, bombs, boss HP multiplier), `KILLS_PER_STAGE` (18), `THEMES` (6 sky palettes, cycled per stage), `ENEMY_TYPES` (grunt, interceptor, tank), `BOSSES` (6 bosses with colors and HP multipliers).
- **Audio**: lazily created `AudioContext` via `ensureAudio()`; sound effects are synthesized with `beep()` and `noiseBurst()`. `muted` gates output.
- **Input**: keyboard (`keys[e.code]`), touch joystick (`joystickStart/Move/End`) with FIRE and BOMB buttons, and mouse (`canvasPointFromEvent`).
- **Layout**: `BASE_W = 480`, `BASE_H = 720` logical resolution; `resize()` scales the canvas.
- **Game state**: `let` globals (`score`, `lives`, `stage`, `chain`, `multiplier`, `player`, and entity arrays). `resetGame()` reinitializes them.
- **Simulation**: `update(dt)` drives movement, firing (`fireWeapon()`), spawning (`spawnEnemy()`, `spawnBoss()`, `maybeSpawnPowerUp()`), collisions (`rectsOverlap`), and scoring (`addScore`, combo `chain` / `multiplier`, `breakChain`).
- **Weapons and power-ups**: weapons are `vulcan` and `laser`, each with levels 1–3 (`applyPowerUp`). Power-up types: `life`, `bomb`, `weapon`.
- **Bosses**: `BOSS_PATTERNS` is an array of bullet-pattern functions (radial, spread, spiral, cross, homing, etc.) used by `updateBoss(dt)`; `killBoss()` advances the stage.
- **Lifecycle**: `loseLife()`, `endGame()`, `togglePause()`.
- **Rendering**: `draw()` plus helpers `drawPlane`, `drawBossShip`, `drawClouds`. `quality` (0 full, 1 medium, 2 low) is adjusted by `trackPerf(dt)` to keep the frame rate up.
- **Main loop**: `loop(now)` via `requestAnimationFrame`.

## Conventions

- Keep everything in `index.html` unless there's a clear reason to split it; the project is intentionally dependency-free and opens straight from the file system.
- No external scripts or stylesheets. Fonts fall back to system stacks (`'Segoe UI', Roboto, Arial`).
- Use `const` for tuning tables and `let` for runtime state; match the existing 2-space indentation inside the script.
- Preserve the mobile/touch support: `touch-action: none` on the canvas and controls, `preventDefault` on touch handlers, and the `viewport-fit=cover` / safe-area insets in CSS.
- Any new persistent setting should use a `skyfighter.*` localStorage key and be wrapped so the game still runs if storage is unavailable.
- Accessibility: announcements go through `announce()` into the `#srLive` live region; respect the reduced-motion toggle (`reducedMotion`) for shake, flash, and pulse effects.
- Keep gameplay tuning in the config tables (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `BOSS_PATTERNS`) rather than hard-coding numbers in the update loop.

## Controls (for reference when testing)

| Action        | Keyboard                         | Touch            |
|---------------|----------------------------------|------------------|
| Move          | Arrow keys or WASD               | Left-side joystick |
| Fire          | Space (hold)                     | FIRE button      |
| Bomb          | X or Shift                       | BOMB button      |
| Pause         | P or Escape                      | Pause button     |

Mute and reduced-motion toggles are in the top bar.

## Git workflow

- Develop on the branch assigned for the session; commit with clear messages and push with `git push -u origin <branch>`.
- Do not open pull requests unless explicitly asked.
