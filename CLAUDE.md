# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Google_4 is a small browser game, **Sky Fighter**: a Raiden-style vertical shoot-'em-up. The whole game lives in one file, `index.html` (~1,670 lines), with inline `<style>` and an inline `<script>`. There is no build step, no bundler, no package manifest, and no test suite. `README.md` is a placeholder.

## Running the game

- Open `index.html` directly in a browser, or serve the directory with any static server (for example `python3 -m http.server` and browse to `index.html`).
- There are no install, build, lint, or test commands. Verify changes by loading the page and playing it. Check both desktop (keyboard and mouse) and touch layouts, since the game has separate control schemes for each.

## Architecture

Everything is in `index.html`, in this order:

1. **`<head>`**: meta tags (viewport locks zoom and enables `viewport-fit=cover`), an inline SVG favicon, and the `<title>`.
2. **`<style>`**: dark-only (`color-scheme: dark`), with HUD, overlay card, boss bar, touch controls, and a stage-banner/flash/vignette layer.
3. **`<body>`**: a `#container` holding the `<canvas id="gameCanvas">`, the HUD (`#hud`), boss bar, combo/stage popups, top-bar buttons (mute, reduced motion, pause), touch controls (`#touchControls`), and the start/pause `#overlay` with difficulty buttons (`.diffBtn`).
4. **`<script>`** (a single `'use strict'` IIFE of `const`/`let`/`function` declarations; no modules, nothing exported to `window`):
   - **DOM refs**: every element is grabbed once at the top with `getElementById`.
   - **Tuning data**: `DIFFICULTIES` (spawn rates, enemy speed and fire chance, lives, bombs, boss HP multiplier), `KILLS_PER_STAGE`, `THEMES` (sky palettes per stage, cycled), `ENEMY_TYPES` (`grunt`, `interceptor`, `tank`), and `BOSSES` (six bosses, each with an `hpMul`).
   - **Persistence**: `STORAGE_*` keys in `localStorage` (see below).
   - **Audio**: a lazily created `AudioContext` (`ensureAudio`) with synthesized `beep` and `noiseBurst` sounds routed through a compressor. Sound effects live in the `sfx` table.
   - **Accessibility and perf**: a reduced-motion toggle (also honors `prefers-reduced-motion`), an ARIA live region (`announce`), and `trackPerf`, which adjusts a `quality` level (0 full, 1 medium, 2 low) from average frame time.
   - **Input**: keyboard (`keys` map keyed by `e.code`), mouse steering and click-to-fire on the canvas, and a touch joystick plus FIRE/BOMB buttons.
   - **Game state**: module-level `let`s (`player`, `bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`, `boss`, `score`, `lives`, `bombCount`, `stage`, `chain`, `multiplier`, and so on). `resetGame()` initializes them for a new run.
   - **Simulation**: `update(dt)` advances everything, `spawnEnemy`/`spawnBoss`/`spawnExplosion` create entities, `fireWeapon`, `useBomb`, and `updateBoss` with pattern helpers (`fireRadial`, `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`, `fireCross`, `fireHomingSeeded`, `fireDoubleRing`).
   - **Rendering**: `draw()` renders each frame to the canvas (`drawPlane`, `drawBossShip`, `drawClouds`). `resize()` scales the 480×720 logical canvas to fit the window, capped at 1.6×.
   - **Loop**: `loop(now)` runs on `requestAnimationFrame` and calls `update` then `draw`. `endGame()` and `togglePause()` handle run lifecycle.

### Game rules worth knowing

- After `KILLS_PER_STAGE` (18) kills in a stage, a boss spawns about 1.2 s later. Each boss is an entry in `BOSSES`, and the boss bar shows its name and HP. The stage counter (`stage++`) advances after the boss is defeated.
- Power-ups drop as `life`, `bomb`, or `weapon`. Weapons are `VULCAN` (default) and `laser`, with a level shown in the HUD.
- Kills build a combo `chain`. `multiplier` is `1 + min(9, floor(chain / 5))`, and the chain breaks when `chainTimer` (2.2 s, refreshed on each kill) runs out.

## Persistence

These `localStorage` keys are the only persisted state. Keep them stable, since changing a name silently resets players' saves:

| Key | Holds |
| --- | --- |
| `skyfighter.bestScore` | Best score (number) |
| `skyfighter.muted` | `'1'` when muted |
| `skyfighter.difficulty` | `easy`, `normal`, or `hard` |
| `skyfighter.reducedMotion` | `'1'` or `'0'`; if unset, falls back to `prefers-reduced-motion` |

Storage access is not wrapped in `try/catch` in the current code. If you add more storage calls, consider guarding them, because private browsing or blocked storage can throw.

## Conventions

- **Match the existing style**: 2-space indentation, single quotes in JavaScript, `const` by default, terse one-line object literals for data tables, and short `//` comments only where the intent isn't obvious.
- **Keep it one file.** Don't split the game into modules or add a build step unless the user asks for it. New tuning values belong in the existing data tables (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `THEMES`) rather than as magic numbers in the loop.
- **Keep the logical resolution at 480×720.** Coordinates in the simulation and drawing code assume it; `resize()` handles scaling.
- **Respect the reduced-motion and quality settings** when adding screen shake, flashes, or particle-heavy effects. Gate them the same way existing effects are gated.
- **Keep the controls text in sync.** The `#controlsHint` paragraph and the touch layout describe the controls; update them if you change bindings.
- **Keep the on-screen text accessible.** Announce important game events through `announce()` so screen readers get them.

## Git

- Development happens on feature branches named `claude/...`. The default branch is `main`.
- Commit with clear, descriptive messages. Don't open a pull request unless asked.
