# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Google_4 is a single-page browser game, **Sky Fighter**: a Raiden-style vertical shoot-'em-up rendered on an HTML `<canvas>`.

The whole game lives in one file, `index.html` (~1,670 lines): inline `<style>`, the markup for the HUD and overlays, and one inline `<script>` block. There is no build step, no bundler, no package manager, no test suite, and no external dependencies. The only other tracked file is `README.md`.

## Repository layout

| Path | Purpose |
| --- | --- |
| `index.html` | The entire game: CSS, markup, and JavaScript |
| `README.md` | Minimal project readme |
| `CLAUDE.md` | This file |

## Running the game

No build is needed. Open `index.html` directly in a browser, or serve the directory with any static file server (for example `python3 -m http.server`) and browse to it.

## Code map (inside `index.html`)

The script is one `(function () { 'use strict'; … })();` block that grabs DOM elements at the top (`canvas`, `overlay`, `hud` elements, touch controls, and so on), then defines the game. Key sections, roughly in file order:

- **Storage keys**: `STORAGE_BEST`, `STORAGE_MUTE`, `STORAGE_DIFF`, `STORAGE_REDUCED` (all prefixed `skyfighter.`). These are the only persisted state.
- **Accessibility and perf**: `announce()` writes to the live region; `trackPerf()` adjusts a `quality` level (0 full, 1 medium, 2 low) from frame times.
- **Tuning data**:
  - `DIFFICULTIES`: spawn rate, enemy speed, enemy fire chance, starting lives and bombs, boss HP multiplier.
  - `KILLS_PER_STAGE` (18): kills needed to trigger a boss.
  - `THEMES`: six background palettes, cycled by stage.
  - `ENEMY_TYPES`: `grunt`, `interceptor`, and `tank`, each with HP, speed, score, and the stage it first appears in.
  - `BOSSES`: six boss designs (VANGUARD, BEHEMOTH, SPECTER, DREADNOUGHT, WRAITH, TITAN), cycled by stage.
- **Audio**: a lazily created `AudioContext` (`ensureAudio()`), with `beep()` and `noiseBurst()` primitives and a `sfx` table of named sounds. Sound is muted by the toggle and remembered in storage.
- **Rendering**: `drawPlane()`, `drawBossShip()`, and the background/particle drawing. Canvas is a fixed `BASE_W × BASE_H` (480 × 720) logical size, scaled to fit the window in `resize()`.
- **Input**: `keys` (keyboard state by `e.code`), joystick and fire/bomb buttons for touch, and mouse steering and click-to-fire on the canvas.
- **Game state**: `player`, `bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`, `boss`, plus score, lives, bombs, chain and multiplier, and stage counters. `resetGame()` initializes them.
- **Game loop**: `update(dt)` handles movement, firing, spawning, collisions, and stage progression. `updateBoss(dt)` drives boss behavior and telegraphs. `loop(now)` is the `requestAnimationFrame` driver. `endGame()` and `togglePause()` handle state transitions.
- **Power-ups**: `maybeSpawnPowerUp()`. The weapon power-up toggles between `vulcan` and `laser` on the player.

## Controls (keep these in sync with the on-screen hint)

- Mouse: move to steer, click to fire.
- Keyboard: arrows or WASD to move, Space to fire, X or Shift to drop a smart bomb, P or Esc to pause.
- Touch: drag left to fly (joystick), tap FIRE, tap BOMB.

The on-screen hint is the `#controlsHint` paragraph in the overlay markup. If you change a binding, update that paragraph too.

## Conventions

- **Keep it one file.** Do not split the game into modules or add a build step unless the user asks for it. The single-file, no-dependency layout is intentional.
- **Preserve the tuning tables.** Balance changes belong in `DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, and `THEMES`, not scattered through the update logic.
- **Respect reduced motion.** New visual effects should check the `reducedMotion` flag and the `quality` level, as the existing effects do.
- **Keep mobile working.** Layout uses `clamp()` and `env(safe-area-inset-*)`, and touch controls are always present. Test changes at phone width.
- **Announce important state changes** through `announce()` so screen-reader users get the same information as sighted players (stage changes, boss defeat, pause, and so on).
- **Store only what is already stored.** Persisted state uses the `skyfighter.*` keys above. Add a new key with the same prefix if one is needed.

## Verifying changes

There are no automated tests or linters. To verify a change:

1. Open `index.html` in a browser (or serve it locally) and check the browser console for errors.
2. Play through at least one stage transition and one boss fight, since most logic is coupled to the loop.
3. Check pause, mute, reduced-motion, and difficulty toggles still work.
4. For layout changes, check both a desktop viewport and a phone-sized viewport.

If you add tooling later (tests, a linter, a build), document the commands here.

## Git

- The default development branch for automated work is `claude/gifted-tesla-v9hs9i`. Do not push to other branches without explicit permission.
- Do not open pull requests unless the user asks for one.
