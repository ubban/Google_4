# CLAUDE.md

Guidance for Claude Code when working in this repository.

## What this repo is

A static, dependency-free browser game, **Sky Fighter**: a Raiden-style vertical shoot-'em-up. The whole game lives in a single file, `index.html` (~1,670 lines: inline `<style>`, markup, and one inline `<script>` wrapped in an IIFE with `'use strict'`).

`README.md` is a stub and is not a source of truth for the project.

There is no build step, no package manager, no test suite, no linter, and no CI configuration. Do not invent one unless asked.

## Running it

Open `index.html` directly in a browser, or serve the directory with any static server, for example:

```
python3 -m http.server 8000
```

Then visit `http://localhost:8000/`.

## Verifying changes

Because there are no automated tests, verify changes by running the game in a browser and playing the affected path. Check at least:

- Desktop: keyboard controls, mouse steering and click-to-fire, pause.
- Mobile layout (narrow viewport): touch joystick, FIRE and BOMB buttons, no horizontal page scroll.
- Any changed stage, enemy, boss, or power-up path.

Syntax-check the inline script after edits, for example by extracting it and running `node --check`, or by loading the page and watching the console for errors.

## Code layout inside `index.html`

Top to bottom:

1. **`<head>`**: meta tags, an inline SVG favicon, and the `<style>` block. The CSS uses custom properties and includes a `.reduced-motion` body class and a `.sr-only` utility.
2. **Markup**: `#container` holds the `#gameCanvas` canvas, a vignette and flash overlay, the HUD (`#hud`, `#topBar`, `#bossBarWrap`, `#comboPop`, `#stageBanner`), touch controls (`#touchControls`), and the start/pause overlay (`#overlay`, `#overlayCard`, `#diffRow`, `#statsRow`).
3. **Script** (single IIFE). Sections, in order, marked by `// ---------- Section ----------` comments:
   - DOM references and `STORAGE_*` keys.
   - Adaptive performance: `quality` (0 full, 1 medium, 2 low) is nudged by `trackPerf` based on a 45-frame rolling average.
   - Tunable data tables: `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES` (sky palettes per stage), `ENEMY_TYPES`, `BOSSES`.
   - Persisted settings: best score, mute, difficulty, reduced motion.
   - Audio: `ensureAudio`, `beep`, `noiseBurst`, and the `sfx` object. Audio is synthesized with WebAudio; there are no audio files.
   - HUD and feedback helpers: hearts, bombs, flashes, combo pop, stage banner, screen shake.
   - Canvas sizing: `BASE_W = 480`, `BASE_H = 720`, scaled to the window (max scale 1.6).
   - Input: keyboard (`keys` map keyed by `e.code`), touch joystick and buttons, mouse.
   - Game state (`score`, `lives`, `bombCount`, `stage`, `chain`, `multiplier`, and the entity arrays) and `resetGame`.
   - Drawing helpers: `drawPlane`, `drawBossShip`, `drawClouds`.
   - Spawning: `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`, `spawnExplosion`.
   - Combat: `fireWeapon`, `useBomb`, scoring (`addScore`, `breakChain`), `applyPowerUp`.
   - Boss attacks: `BOSS_PATTERNS` (an array of pattern functions that push into `enemyBullets`) plus the `fire*` helpers, `updateBoss`, `killBoss`.
   - Flow: `loseLife`, `endGame`, `togglePause`.
   - Rendering: `draw()`, and the main `update(dt)` / `loop(now)` game loop driven by `requestAnimationFrame`. `dt` is clamped to 50 ms.

## Conventions and gotchas

- **Keep it one file.** New code goes in `index.html` unless the user asks for a split. Do not add a bundler, framework, or npm dependency.
- **Match the existing style.** Two-space indentation, single quotes in JS, `const`/`let` (no `var`), arrow functions for callbacks, and short section banners. Comments are sparse and explain *why*, not *what*.
- **Data-driven tuning.** Balance changes belong in the tables (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `THEMES`), not scattered as magic numbers in logic.
- **Entities are plain objects in arrays** (`bullets`, `enemies`, `enemyBullets`, `powerUps`, and so on). Remove dead entries by iterating backwards and splicing, as the existing loops do.
- **Pause and game state.** Gameplay runs only when `running && !paused`. Anything that should freeze on pause must respect that guard.
- **Persistence uses `localStorage` with keys prefixed `skyfighter.`.** Keep that prefix for any new keys. Reads and writes are not wrapped in try/catch today; if you add storage access, guard it so the game still works when storage is blocked.
- **Accessibility.** Screen-reader announcements go through `announce()` into `#srLive`. Respect the reduced-motion setting (`reducedMotion`, the `.reduced-motion` class) for any new animation or shake.
- **Mobile.** The layout must work at phone width without horizontal scroll. Touch and mouse both drive `firing`. The bomb is triggered by `X`/`Shift` on the keyboard or the BOMB button on touch screens.
- **Controls** (also shown in `#controlsHint`): arrows/WASD move, Space fires (or hold mouse/FIRE), X or Shift bombs, P or Esc pauses. If you change a binding, update the hint text too.
- **Keep the page self-contained.** No external scripts, fonts, or network calls. The favicon is an inline data URI.

## Git

- Development branch for this workspace: `claude/gifted-tesla-qienh8`. Commit and push there.
- Commit messages describe the change in the imperative mood, in the style of the existing history (for example, `Add Sky Fighter: Raiden-style vertical shoot-em-up game`).
