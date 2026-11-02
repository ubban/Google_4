# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a single-page browser game, **Sky Fighter** — a Raiden-style vertical shoot-'em-up with weapon switching, smart bombs, power-ups, and stage bosses.

The entire game lives in one file, `index.html` (~1,670 lines): inline CSS in `<style>`, the canvas/HUD markup, and all game logic in one inline `<script>`. There is no build step, no bundler, no `package.json`, no test suite, and no external dependencies.

`README.md` is a stub (just the project title) and is not a source of truth for behavior.

## Running the game

Open `index.html` directly in a browser, or serve the directory with any static server, for example:

```sh
python3 -m http.server 8000   # then visit http://localhost:8000
```

There is nothing to install or compile.

## Code layout inside `index.html`

Approximate landmarks (line numbers drift as the file changes; search by name):

| Area | What's there |
| --- | --- |
| `<style>` (top of file) | Layout, HUD styling, hearts/bombs icons, vignette, animations |
| Markup after `<style>` | `#container`, `#gameCanvas`, `#hud`, mobile controls, `#controlsHint` |
| Constants (`STORAGE_*`, `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES`, `ENEMY_TYPES`, `BOSSES`) | Tuning values and stage/boss definitions |
| Persistence | `localStorage` keys `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion` |
| Audio | `ensureAudio`, `beep`, `noiseBurst` — WebAudio synthesized sound effects (no audio files) |
| HUD/feedback | `renderHearts`, `renderBombs`, `flashHit`, `showComboPop`, `showStageBanner`, `screenShake` |
| Input | Keyboard (`keydown`/`keyup`), touch joystick + fire/bomb buttons, mouse steering on the canvas |
| Canvas setup | `BASE_W = 480`, `BASE_H = 720`, `resize()` |
| Game state | `resetGame()` and the global `let` state (score, lives, bombs, stage, chain, multiplier) |
| Entities & spawning | `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`, `spawnExplosion` |
| Weapons | `fireWeapon` (vulcan/other weapons by `player.weapon` and `player.level`), `useBomb` |
| Update loop | `update(dt)`, `applyPowerUp`, `updateBoss`, `killBoss`, `loseLife`, `endGame`, `togglePause` |
| Boss patterns | `BOSS_PATTERNS` and the `fire*` helpers (radial, spread, spiral, cross, homing, double ring) |
| Rendering | `drawPlane`, `drawBossShip`, `drawClouds`, `draw()` |
| Main loop | `loop(now)` — `requestAnimationFrame`-driven, with `trackPerf(dt)` adjusting `quality` (0 full / 1 medium / 2 low) |

## Game rules and controls (for reference when changing behavior)

- **Difficulty** (`DIFFICULTIES`): `easy`, `normal` (default), `hard`. They differ in spawn rate, enemy speed, enemy shot chance, starting lives, starting bombs, and boss HP multiplier.
- **Stages**: advance after `KILLS_PER_STAGE` (18) kills; each stage has a boss from `BOSSES`.
- **Lives and bombs**: `lives` and `bombs` come from the difficulty setting; `maxBombs` caps the bomb count.
- **Controls**:
  - Keyboard: arrows or WASD to move; Space to fire; `X` or Shift for bomb; `P` or Esc to pause.
  - Mouse: move to steer, click to fire.
  - Touch: drag on the left zone to fly, tap FIRE, tap BOMB.
- The `controlsHint` text in the markup documents these controls. Update it if you change bindings.

## Conventions

- **Keep it a single file.** Don't split out modules, add a build step, or introduce dependencies unless the user asks for that change.
- **Vanilla JS only.** No frameworks. Follow the existing style: `const`/`let`, arrow functions for handlers, small named helper functions.
- **Frame-rate independence.** Movement and timers use `dt` (seconds) from `loop`. Keep new motion/timer code in that style rather than per-frame increments.
- **Mobile-first layout.** The page uses `touch-action: none`, safe-area insets, and `clamp()` sizing. Check that any HUD or UI change still works at phone width without page scroll.
- **Accessibility and comfort.** A reduced-motion setting exists (`skyfighter.reducedMotion`) and an `announce()` helper for screen readers. Respect both when adding effects or messages.
- **Performance.** `trackPerf` lowers `quality` when frames are slow. Gate expensive effects on `quality` where practical.
- **Persistence.** Use the existing `STORAGE_*` keys pattern and wrap storage access in a way that tolerates unavailable `localStorage` if you add new keys.

## Testing and verification

There are no automated tests or linters in this repo. Verify changes by:

1. Loading `index.html` in a browser and playing a few stages.
2. Checking all three difficulties, keyboard, mouse, and (if relevant) touch input.
3. Confirming the HUD and controls hint stay correct at both desktop and phone-sized viewports.
4. Opening the browser console and confirming there are no errors during play, pause, bomb use, boss fights, and game over.

If you add non-trivial logic, consider keeping the pure calculations separate and easy to check by hand.

## Git workflow

- Develop on the branch the task names (currently `claude/gifted-tesla-6e61at`). Don't push to other branches without explicit permission.
- Write clear commit messages describing the gameplay or code change.
- Don't open a pull request unless the user asks for one.
