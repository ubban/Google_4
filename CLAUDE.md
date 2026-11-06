# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a small static web project. Its only app is **Sky Fighter**, a
Raiden-style vertical shoot-'em-up written as a single self-contained HTML5
canvas page.

| File        | Purpose                                                        |
|-------------|----------------------------------------------------------------|
| `index.html`| The entire game: markup (~360 lines of CSS, HUD/overlay DOM), and one inline `<script>` IIFE (~1,250 lines) |
| `README.md` | Minimal project readme                                          |

There is no build step, package manager, test suite, or linter configuration.
Nothing needs to be installed.

## Running the game

Open `index.html` directly in a browser, or serve the folder with any static
server (e.g. `python3 -m http.server`). Do not add a bundler or framework unless
the user asks for one.

## Code structure of `index.html`

- **CSS (`<style>`)**: HUD pills, glassmorphic overlay card, touch controls,
  vignette/flash effects, and a `.reduced-motion` body class.
- **DOM**: `#gameCanvas`, the `#overlay` start/pause/game-over card, HUD
  elements (`#score`, `#best`, `#mult`, `#weaponName`, `#hearts`, `#bombs`,
  `#bossBarWrap`), touch controls (`#touchControls`, `#moveZone`, `#fireBtn`,
  `#bombBtn`), and an `aria-live` region (`#srLive`) for screen readers.
- **Script**: one IIFE with `'use strict'`. Everything lives inside it, so
  nothing is exposed on `window`. Keep new code inside the IIFE.

Key sections inside the script, in order:

1. **Config constants**: `STORAGE_*` keys, `DIFFICULTIES`, `KILLS_PER_STAGE`
   (18), `THEMES` (6 sky palettes cycled by stage), `ENEMY_TYPES`
   (`grunt`, `interceptor`, `tank`), and `BOSSES` (6 bosses with `hpMul`).
2. **Adaptive quality**: `trackPerf()` samples frame times and steps a
   `quality` level 0–2 to reduce effects on slow devices.
3. **Audio**: a Web Audio synth (`beep`, `noiseBurst`, and a `sfx` object).
   Audio is created lazily by `ensureAudio()`, which browsers require to run
   after a user gesture.
4. **Input**: keyboard, a touch joystick, touch/mouse fire and bomb buttons,
   and mouse aim on the canvas.
5. **Game state and logic**: `resetGame`, `update(dt)`, `spawnEnemy`,
   `spawnBoss`, `applyPowerUp`, `useBomb`, `fireWeapon`, `loseLife`, `endGame`,
   `togglePause`.
6. **Boss patterns**: `BOSS_PATTERNS` and the `fire*` helpers
   (`fireRadial`, `fireAimedSpread`, `fireSpiralBurst`, etc.) drive `updateBoss`.
7. **Rendering**: `draw()` and its helpers (`drawPlane`, `drawBossShip`,
   `drawClouds`). Drawing is canvas 2D only.
8. **Main loop**: `loop(now)` runs on `requestAnimationFrame`. The start button
   wires up the first `resetGame()`.

## Game rules worth knowing

- Three difficulties (`easy`, `normal`, `hard`) change spawn rate, enemy speed,
  enemy shot chance, starting lives, starting bombs, and boss HP multiplier.
- Weapons: `vulcan` and `laser`, each with 3 power levels. A weapon power-up
  raises the level, and past level 3 it switches weapon and resets to level 1.
- Power-ups: `life`, `bomb`, and `weapon`.
- Bombs clear enemy bullets and enemies, and deal 24 damage to the boss.
- A kill chain builds the score multiplier (`mult`). Getting hit breaks it.
- Each stage ends after `KILLS_PER_STAGE` (18) stage kills, which triggers a
  boss (`spawnBoss`, selected as `BOSSES[(stage - 1) % 6]`, HP =
  `(70 + stage × 35) × hpMul × difficulty.bossHpMul`). Boss pattern and theme
  follow the stage number.

## Persistence

`localStorage` keys (all prefixed `skyfighter.`):

- `bestScore`: the high score
- `muted`: `'1'` when audio is muted
- `difficulty`: the last chosen difficulty (defaults to `normal`)
- `reducedMotion`: `'1'` or `'0'`. If unset, it follows
  `prefers-reduced-motion`.

Every `localStorage` access should stay simple. If you add a new key, use the
`skyfighter.` prefix and document it here.

## Conventions

- **Single file**: keep the game in `index.html` unless the user asks for a
  split. Do not add a build system for this.
- **Vanilla JS, ES2017+**: use `const`/`let`, arrow functions, and template
  strings as the existing code does. No external libraries or CDN links.
- **Accessibility**: respect `reducedMotion`, keep the `aria-live` announcements
  (`announce()`) for important events, and keep keyboard and touch controls
  working together.
- **Mobile**: layouts scale through `resize()` with a fixed `BASE_W × BASE_H`
  (480 × 720) logical canvas. Test changes to HUD or touch controls at phone
  widths.
- **Performance**: avoid per-frame allocations in `update`/`draw` where easy,
  and respect the `quality` level when adding visual effects.

## Verifying changes

There are no automated tests. For changes to the game:

1. Open `index.html` in a browser (or serve it statically) and check the
   console for errors.
2. Play through: start, move, fire, use a bomb, collect power-ups, reach a
   boss, pause and resume, and lose all lives to reach game over.
3. Check the HUD at desktop and phone widths, and the touch controls if
   changed.
4. If you have a browser automation tool available, a quick headless run that
   loads the page and checks for console errors is a good smoke test.

## Git workflow

- Develop on the branch the session specifies. Do not push to other branches
  without permission.
- Write clear commit messages that describe the gameplay or code change.
- Do not open a pull request unless the user asks for one.
