# CLAUDE.md

Guidance for Claude Code when working in this repository.

## What this repo is

A small static web project. The only real content is **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs in a single HTML5 `<canvas>` page.

| Path | Purpose |
| --- | --- |
| `index.html` | The entire game: markup, CSS, and JavaScript in one file (~1,670 lines) |
| `README.md` | Title only |

There is no build step, no package manager, no test suite, and no dependencies. Do not add any unless asked.

## Running it

Open `index.html` directly in a browser, or serve the folder with any static server (for example `python3 -m http.server`). Nothing needs to be installed.

## Structure of `index.html`

- **`<style>` (lines ~10–359):** all CSS, including the HUD pill bar, glassmorphic overlay card, touch controls, boss bar, and the `.reduced-motion` class.
- **`<body>` (lines ~361–420):** DOM for the canvas, HUD, boss bar, combo and stage banners, top-right icon buttons (mute, reduced motion, pause), touch controls (joystick, FIRE, BOMB), and the start/pause overlay card.
- **`<script>` (lines ~422–1667):** one IIFE wrapped in `'use strict'`. Nothing is exposed on `window`. Keep new code inside this IIFE.

Within the script, the sections run in this order:

1. DOM lookups and `STORAGE_*` keys
2. `announce()` for screen-reader updates and `trackPerf()` for adaptive quality (0 full, 1 medium, 2 low)
3. Config tables: `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES`, `ENEMY_TYPES`, `BOSSES`
4. Settings: mute, reduced motion, difficulty (each persisted to `localStorage`)
5. Audio: `ensureAudio()`, `beep()`, `noiseBurst()`, and explosion SFX. Audio is synthesized with WebAudio, so there are no sound files.
6. HUD and effects helpers: hearts, bombs, combo pop, stage banner, screen shake, hit flash
7. Input: keyboard, mouse, pointer, and joystick handling, plus `resize()` (base resolution 480×720, scaled to fit)
8. Game state and logic: `resetGame()`, `spawnEnemy()`, `spawnBoss()`, `update(dt)`, `fireWeapon()`, `useBomb()`, `applyPowerUp()`, `loseLife()`, `endGame()`, `togglePause()`
9. Boss patterns: `BOSS_PATTERNS` (radial, aimed spread, columns, spiral, cross, homing, double ring) driven by `updateBoss(dt)`
10. Rendering: `drawPlane()`, `drawBossShip()`, `drawClouds()`, `draw()`
11. `loop(now)` runs on `requestAnimationFrame`. `dt` is clamped to 50 ms. Updates run only when `running && !paused`.

## Game rules (current values)

- **Lives and bombs by difficulty:** Easy 4 lives / 3 bombs, Normal 3 / 2, Hard 3 / 2. Hard also raises enemy speed, shot frequency, and boss HP.
- **Stages:** 18 kills per stage, then a boss fight. Six boss designs and six background themes cycle with stage number (`(stage - 1) % 6`).
- **Enemies:** `grunt` (1 HP), `interceptor` (weaving, fast), `tank` (4 HP, from stage 2).
- **Weapons:** Vulcan and Laser, each with 3 power levels, upgraded by power-ups.
- **Scoring:** kill-chain multiplier shown in the HUD. Score and best score are tracked.
- **Power-ups:** weapon level, bomb, extra life.

## Controls

- **Desktop:** mouse moves and fires (click); arrows or WASD move; Space fires; X or Shift triggers a bomb; P or Esc pauses.
- **Touch:** drag left side to fly, tap FIRE, tap BOMB.
- Space's default action is always prevented, so the page does not scroll.

## Persistence

All saved state uses `localStorage` with the `skyfighter.` prefix:

| Key | Value |
| --- | --- |
| `skyfighter.bestScore` | number |
| `skyfighter.muted` | `'1'` or `'0'` |
| `skyfighter.difficulty` | `easy`, `normal`, or `hard` |
| `skyfighter.reducedMotion` | `'1'` or `'0'`. If unset, defaults to the `prefers-reduced-motion` media query. |

Renaming a key resets player data, so change it only on purpose.

## Conventions

- **Single file:** keep the game in `index.html` unless the user asks for a split. The project has no bundler.
- **Match the existing style:** 2-space indentation, single quotes in JS, `const` by default, arrow functions for callbacks, and `// ----------` banner comments to group sections.
- **Config over magic numbers:** tuning values go in the existing config tables (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `THEMES`) rather than inline literals.
- **Accessibility:** announce important state changes through `announce()` so they reach `#srLive`. Respect the reduced-motion flag for shake, flash, and pulse effects.
- **Performance:** rendering adapts through `quality`. Avoid per-frame allocations in `update` and `draw` where practical.
- **Pure canvas and DOM:** no external libraries, fonts, or network calls.

## Verifying changes

There are no automated tests or linters. To verify a change:

1. Open `index.html` in a browser (or run a static server) and check the browser console for errors.
2. Play through at least one stage and one boss fight on the affected difficulty.
3. Check the touch layout at phone width if you changed HUD or controls, since the UI has separate mobile rules.
4. Confirm that the pause, mute, and reduced-motion toggles still persist across a reload.

## Git

- Develop on the branch assigned to the session. Do not push to other branches without explicit permission.
- Commit messages are imperative and descriptive, with a short summary line and a body explaining the change, in the style of the existing history (`Add Sky Fighter: ...`).
- Do not open a pull request unless the user asks for one.
