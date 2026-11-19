# CLAUDE.md

Guidance for Claude Code when working in this repository.

## What this project is

**Sky Fighter** — a Raiden-style vertical shoot-'em-up that runs in the browser on an HTML5 `<canvas>`. The whole game is one self-contained file, `index.html` (~1,670 lines: inline `<style>`, markup, and one inline `<script>`). There is no framework, no bundler, no package manager, and no external dependencies.

Other files:
- `README.md` — a stub (title plus a timestamp line). Not documentation of the game.

There is no test suite, linter, or build step in the repo.

## Running it

Open `index.html` directly in a browser, or serve the directory with any static server, for example:

```bash
python3 -m http.server 8000   # then visit http://localhost:8000/
```

## Code layout inside `index.html`

The script is a single IIFE (`(function () { 'use strict'; ... })();`). Sections, in order:

| Section | Purpose |
|---|---|
| DOM references | `const xxxEl = document.getElementById(...)` at the top |
| Storage keys | `skyfighter.bestScore`, `.muted`, `.difficulty`, `.reducedMotion` (localStorage) |
| Adaptive performance | `trackPerf` lowers/raises a `quality` level (0–2) from a 45-frame average |
| Tuning tables | `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES` (per-stage sky palettes), `ENEMY_TYPES`, `BOSSES` |
| Audio | `ensureAudio`, `beep`, `noiseBurst`, and the `sfx` object (Web Audio, no asset files) |
| HUD helpers | `renderHearts`, `renderBombs`, `flashHit`, `showComboPop`, `showStageBanner`, `screenShake` |
| Sizing | `BASE_W = 480`, `BASE_H = 720`; `resize()` scales the canvas to fit the window (max 1.6×) |
| Input | Keyboard (`keys[]`), mouse (`mouseActive`), touch joystick and buttons |
| Game state | `resetGame()` initialises `player`, `bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`, `boss` |
| Drawing | `drawPlane`, `drawBossShip`, `drawClouds`, `draw()` |
| Spawning | `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp` |
| Combat | `fireWeapon`, `applyPowerUp`, `BOSS_PATTERNS` (radial, spread, spiral, cross, homing, double-ring), `updateBoss`, `killBoss` |
| Game flow | `loseLife`, `endGame`, `togglePause`, `useBomb`, `breakChain` |
| Main loop | `update(dt)` → `draw()` → `requestAnimationFrame(loop)`; `dt` is clamped to 50 ms |

Game-logic code uses `dt` (seconds) for all movement and timers, so keep new motion frame-rate independent.

## Game rules worth knowing

- **Difficulty** (Easy / Normal / Hard) sets spawn rate, enemy speed, enemy fire chance, starting lives, starting bombs, and boss HP multiplier. The choice is persisted and can only be changed between runs.
- **Stages:** a stage ends after `KILLS_PER_STAGE` (18) kills; a boss spawns 1.2 s later (`spawnBoss`). Stage visuals come from the `THEMES` palette. `BOSSES` holds six bosses with different HP multipliers and attack patterns (`BOSS_PATTERNS`).
- **Enemies:** `grunt`, `interceptor` (weaving), and `tank` (4 HP, unlocked at stage 2). Score values are in `ENEMY_TYPES`.
- **Power-ups** drop randomly from kills (`maybeSpawnPowerUp`): `life` (~6%), `bomb` (~8%), `weapon` (~20%, upgrades the weapon shown in the HUD).
- **Combo multiplier:** each kill increments the chain; the multiplier is `1 + floor(chain / 5)`, capped at 10×. `breakChain()` resets it.
- **Bombs** are triggered by X, Shift, or the BOMB button. They clear enemy bullets and enemies on screen, add score, and deal 24 damage to the boss.

## Controls (must stay in sync with the on-screen hint)

The hint text lives in `#controlsHint` in the markup. If you change a binding, update that string too.

- Keyboard: Arrows / WASD move, Space fire, X or Shift bomb, P or Esc pause.
- Mouse: move to steer, click to fire.
- Touch: drag left zone to fly, FIRE and BOMB buttons on the right.

## Accessibility and preferences

- `reducedMotion` disables screen shake, flashes, and pulses. It defaults to the OS `prefers-reduced-motion` setting and can be toggled with the 🎆 button; the choice is stored.
- `#srLive` is an `aria-live` region used by `announce()` for screen readers. Use `announce()` for important state changes (stage cleared, boss incoming, game over) rather than adding new live regions.
- Keep the page usable at phone widths; touch controls are shown only on touch devices.

## Conventions

- Plain ES2017+ JavaScript, no modules, no transpilation. Keep it runnable by opening the file directly.
- Keep everything in `index.html` unless the file becomes unmanageable. If you split it, move CSS and JS into separate files and update this document.
- Put tunable numbers in the tuning tables at the top of the script rather than inline magic numbers.
- Reuse the existing helpers (`beep`, `noiseBurst`, `spawnExplosion`, `addScore`, `rectsOverlap`, `announce`) instead of duplicating them.
- Use `localStorage` only through the existing `STORAGE_*` keys, and keep the existing key prefix `skyfighter.`.
- Do not add a build step, frameworks, or external CDN assets without a clear reason; the game is intentionally a single portable file.

## Verifying changes

There are no automated tests. For any gameplay or UI change:

1. Open the page in a browser (or use the pre-installed Playwright Chromium) and confirm there are no console errors.
2. Play through at least one stage and one boss on the affected difficulty.
3. Check the touch layout at a phone-sized viewport if you touched controls, HUD, or `resize()`.
4. Check that pause (P/Esc), mute, and reduced-motion toggles still work.

## Git

- Default development branch for this session: `claude/gifted-tesla-kzk5p5`.
- Commit messages should describe the gameplay or code change in the imperative mood.
