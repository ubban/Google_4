# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo contains **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs in a browser on an HTML5 `<canvas>`.

- `index.html` — the entire game (~1,670 lines): CSS in `<style>`, markup in `<body>`, and game code in one inline `<script>` IIFE.
- `README.md` — currently a stub. Keep it in sync if you change how the project is run or shipped.

There is no build step, no package manager, no bundler, no framework, and no external dependencies or CDN scripts. Do not introduce any of these without the user asking for it.

## Running and verifying

- **Run:** open `index.html` directly in a browser. No server is needed.
- **Tests:** there is no test suite.
- **Syntax check** (fast, no browser needed). Extract the inline script and run it through Node:
  ```sh
  awk '/<script>/{f=1;next}/<\/script>/{f=0}f' index.html > "$SCRATCH/game.js" && node --check "$SCRATCH/game.js"
  ```
  Set `SCRATCH` to a scratch directory outside the repo (for example, the session scratchpad).
- **Behavior changes:** verify in a real browser (Playwright is available in cloud sessions). Start a game, check the affected mechanic, and watch the console for errors.

## Architecture

Everything lives inside one IIFE with `'use strict'`. Top-level state is kept in `let` variables inside that closure, not on `window`.

Major sections of the script, in file order:

1. **DOM refs and constants** — element lookups at the top, `STORAGE_*` keys, `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES`, `ENEMY_TYPES`, `BOSSES`.
2. **Adaptive quality** — `trackPerf()` samples frame times and steps `quality` (0 full / 1 medium / 2 low) to keep frame rate up.
3. **Audio** — `ensureAudio()`, `beep()`, `noiseBurst()`, and the `sfx` table. Audio is created lazily on the first user gesture (`ensureAudio()` is called from start, fire, and bomb handlers).
4. **HUD and feedback** — `renderHearts()`, `renderBombs()`, `flashHit()`, `showComboPop()`, `showStageBanner()`, `screenShake()`.
5. **Input** — keyboard (`keydown`/`keyup` on `e.code`), mouse on the canvas, and touch joystick plus FIRE/BOMB buttons.
6. **Game state and lifecycle** — `resetGame()`, `update(dt)`, `endGame()`, `togglePause()`, `loseLife()`, `killBoss()`.
7. **Spawning and combat** — `spawnEnemy()`, `spawnBoss()`, `fireWeapon()`, `BOSS_PATTERNS` (radial, aimed spread, columns, spiral, cross, homing, double ring), `updateBoss()`.
8. **Rendering** — `draw()`, `drawPlane()`, `drawBossShip()`, `drawClouds()`.
9. **Main loop** — `loop(now)` runs on `requestAnimationFrame`, clamps `dt` to 0.05 s, calls `update(dt)` then `draw()`, and only updates while `running && !paused`.

### Game rules worth knowing

- **Canvas** is a logical 480×720 (`BASE_W`/`BASE_H`), scaled to fit the window by `resize()`. Write game coordinates in logical units, not CSS pixels.
- **Difficulty** (`easy` / `normal` / `hard`) sets spawn rate, enemy speed, enemy fire chance, starting lives and bombs, and a boss HP multiplier.
- **Stages:** a stage ends after `KILLS_PER_STAGE` (18) kills, which triggers a boss. Boss HP is `70 + stage × 35`, scaled by the boss's `hpMul` and difficulty. `THEMES` and `BOSSES` cycle with `(stage - 1) % length`.
- **Enemies:** `grunt` and `interceptor` appear from stage 1; `tank` (4 HP) from stage 2. Choose spawns via `minStage`.
- **Weapons:** `vulcan` (spread shot) and `laser` (pierce). Each has power levels 1–3. The weapon power-up raises the level up to 3, then toggles between vulcan and laser at level 1.
- **Power-ups:** `life`, `bomb`, `weapon`. Some destroyed enemies drop one (`maybeSpawnPowerUp()`), and it falls down the screen.
- **Kill chain:** each kill extends a 2.2 s chain timer. The score multiplier is `1 + min(9, floor(chain / 5))`.
- **Bombs** (X, Shift, or the touch BOMB button) clear enemy bullets and enemies on screen, and deal a flat 24 damage to the boss.
- **Persistence:** all saved state goes to `localStorage` under the `skyfighter.*` prefix: `bestScore`, `muted`, `difficulty`, `reducedMotion`.

## Conventions

Match the existing code. Keep the style consistent throughout.

- **Indentation:** 2 spaces. Single quotes in JS and HTML attributes.
- **Syntax:** `const` by default, `let` when reassigned. Arrow functions for callbacks. Template literals are fine; the file already uses string concatenation, so keep to whichever is nearby.
- **Naming:** `camelCase` for functions and variables. `UPPER_SNAKE_CASE` for module-level constants and storage keys. Enemy and boss type keys are lowercase strings.
- **Comments:** section dividers use `// ---------- Name ----------`. Explain non-obvious intent briefly; do not narrate what the code does.
- **Accessibility:** use `announce(text)` for screen-reader updates (it writes to the `srLive` live region). Respect the `reduced-motion` body class for any new animation or screen effect.
- **DOM:** look up elements once at the top of the IIFE, the way the file already does, rather than querying inside hot paths like `update()` or `draw()`.
- **Hot paths:** `update()`, `draw()`, and `loop()` run every frame. Avoid allocations and DOM writes there where practical. Gate DOM updates on actual state changes.
- **Dependencies:** none. Do not add libraries, CDN links, or a build step.

## Known gaps

- `localStorage` reads and writes are not wrapped in `try`/`catch`; only the audio setup is guarded. Storage calls can throw in some private-browsing or blocked-storage setups. If you touch storage code, consider guarding it.
- There are no automated tests. Changes to game logic should be checked by playing them.

## Git workflow

- The default branch is `main`. Feature work happens on a dedicated branch (currently `claude/gifted-tesla-f3zh86`).
- Write clear commit messages that describe the change and the reason for it.
- Do not open pull requests unless the user asks.
