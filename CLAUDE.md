# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Google_4 is a small static web project. Its only real content is **Sky Fighter**, a
Raiden-style vertical shoot-'em-up written as a single-page HTML5 canvas app in
`index.html`. There is no build step, no package manager, no framework, and no
test suite.

- `index.html` (~1,670 lines): all markup, CSS (lines ~10–359), and JavaScript
  (lines ~422–1667, one IIFE with `'use strict'`).
- `README.md`: a stub. Do not treat it as documentation.

## Running and verifying

- Open `index.html` directly in a browser, or serve the directory with any static
  server (for example `python3 -m http.server`).
- There are no automated tests or linters configured. Verify changes by playing
  the game in a browser: start a run, check HUD updates, pause/resume, bombs,
  weapon power-ups, a stage boss, game over, and the best-score save.
- Check mobile layout and touch controls when you touch the HUD, the overlay, or
  the `#touchControls` block. The page sets a `viewport` with `user-scalable=no`
  and uses `BASE_W = 480`, `BASE_H = 720` with `resize()` for scaling.

## Code structure (inside `index.html`)

The script is organized by section comments and top-level `const`/`let` blocks:

| Area | Key names |
| --- | --- |
| Persistence | `STORAGE_BEST`, `STORAGE_MUTE`, `STORAGE_DIFF`, `STORAGE_REDUCED` (all `localStorage`, `skyfighter.*` keys) |
| Difficulty | `DIFFICULTIES` (easy / normal / hard): spawn rates, enemy speed, shoot chance, starting lives and bombs, boss HP multiplier |
| Stage content | `KILLS_PER_STAGE` (18), `THEMES` (sky palettes cycled per stage), `ENEMY_TYPES` (grunt, interceptor, tank), `BOSSES` (six boss designs), `BOSS_PATTERNS` (bullet patterns, `fire*` helpers) |
| Audio | `ensureAudio()`, `beep()`, `noiseBurst()`, `EXPLODE_SFX`. Web Audio is created lazily on the first user gesture. |
| Rendering | `draw()`, `drawPlane()`, `drawBossShip()`, `drawClouds()`, `spawnExplosion()` |
| Simulation | `update(dt)`, `spawnEnemy()`, `spawnBoss()`, `updateBoss(dt)`, `fireWeapon()`, `useBomb()`, `applyPowerUp()`, `maybeSpawnPowerUp()`, `addScore()`, `breakChain()` |
| Flow | `resetGame()`, `loseLife()`, `killBoss()`, `endGame()`, `togglePause()`, `loop(now)` |
| Input | keyboard (`keydown`), mouse on `canvas`, touch joystick (`joystick*`), `fireBtn`, `bombBtn` |
| Adaptive quality | `trackPerf(dt)` lowers or raises `quality` (0 full, 1 medium, 2 low) based on a 45-frame average |
| Accessibility | `announce()` writes to the `#srLive` region. Reduced motion is toggled via `body.reduced-motion` and also honors `prefers-reduced-motion`. |

The main loop is `requestAnimationFrame(loop)`. `dt` is clamped to 50 ms. The
loop calls `update(dt)` and then `draw()` only while the game is running and not
paused.

## Conventions

- Keep the single-file structure. Do not split the app into modules, add a build
  step, or add dependencies unless the user asks for it.
- Match the existing style: 2-space indentation, single quotes in JS, `const` by
  default, arrow functions for callbacks, and short section banner comments such
  as `// ---------- Adaptive performance ----------`.
- Game tuning belongs in the constant tables (`DIFFICULTIES`, `ENEMY_TYPES`,
  `BOSSES`, `THEMES`) rather than hard-coded numbers in the update logic.
- Any new persisted setting should use the `skyfighter.` prefix for its
  `localStorage` key, follow the existing `STORAGE_*` constants, and apply a
  default when the key is missing (see how `STORAGE_DIFF` is read).
- Accessibility and mobile support are part of the feature set. Keep keyboard,
  mouse, and touch input working together, and keep `aria-live` announcements
  for important state changes.

## Git workflow

- Default working branch for this session: `claude/gifted-tesla-umcsgf`.
- Commit with clear, descriptive messages that explain the change.
- Do not open a pull request unless the user explicitly asks for one.
