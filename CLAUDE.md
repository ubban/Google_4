# CLAUDE.md

Guidance for Claude Code when working in this repository.

## What this project is

Sky Fighter is a single-file, Raiden-style vertical shoot-'em-up that runs in the browser on a `<canvas>`. There is no build step, no package manifest, no test suite, and no external dependencies.

Files:

- `index.html` (~1670 lines): the entire game. Inline `<style>` (lines ~10–359), inline markup (~360–421), and one inline `<script>` (~422–1667).
- `README.md`: a one-line title. It is not documentation.

## Running it

Open `index.html` directly in a browser. Nothing needs to be installed or compiled. For a quick check with a local server, any static server works, for example `python3 -m http.server` from the repo root.

There are no automated tests or linters configured. Verify changes by playing the game: start a run, check the affected mechanic, and check the mobile layout if the change touches HUD, overlay, or touch controls.

## Code layout inside `index.html`

The script is a single IIFE-style block with top-level `const`/`let` state and plain functions. Keep new code in the same style.

Rough order of sections, matching the file:

1. **DOM references**: one `const xEl = document.getElementById(...)` per element, grouped at the top.
2. **Storage keys**: `STORAGE_BEST`, `STORAGE_MUTE`, `STORAGE_DIFF`, `STORAGE_REDUCED` (all prefixed `skyfighter.`).
3. **Perf and accessibility**: `trackPerf()` adjusts a `quality` level (0 full, 1 medium, 2 low) from frame times. `announce()` writes to the screen-reader live region.
4. **Tuning tables**: `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES` (sky palettes cycled per stage), `ENEMY_TYPES` (grunt, interceptor, tank), `BOSSES` (six designs, cycled by stage).
5. **Audio**: lazily created `AudioContext` via `ensureAudio()`, with `beep()`, `noiseBurst()`, and a named `sfx` map. Respect `muted`.
6. **HUD helpers**: hearts, bombs, combo pop, stage banner, screen shake, flash.
7. **Input**: `keys` (by `KeyboardEvent.code`), mouse steering and firing, a touch joystick, and FIRE/BOMB buttons.
8. **Game state**: `player`, `bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`, `boss`, plus score, lives, bombs, stage, chain and multiplier. `resetGame()` initializes them.
9. **Gameplay**: `update(dt)` drives movement, firing, spawning, collisions, and power-ups. `fireWeapon()`, `applyPowerUp()`, and `BOSS_PATTERNS` (one array of attack functions per boss) define weapons and boss attacks. `updateBoss()`, `killBoss()`, `loseLife()`, and `endGame()` handle the rest.
10. **Rendering**: `draw()` plus helpers such as `drawPlane()` and `drawBossShip()`. Rendering reads state and does not change it.
11. **Loop and bootstrap**: `loop(now)` runs `update` then `draw` via `requestAnimationFrame`. Event listeners are wired at the bottom.

## Keyboard and controls

- Move: arrow keys or WASD; mouse or touch steers the ship.
- Fire: Space (or hold the left mouse button or FIRE).
- Bomb: X or either Shift.
- Pause: P or Escape.

Keep these in sync with the on-screen `#controlsHint` text.

## Persisted state

`localStorage` keys are the only persistence. Wrap access in `try/catch` if you add more, because private browsing and blocked storage can throw.

| Key | Holds |
| --- | --- |
| `skyfighter.bestScore` | Best score as a number string |
| `skyfighter.muted` | `'1'` when muted |
| `skyfighter.difficulty` | `easy`, `normal`, or `hard` |
| `skyfighter.reducedMotion` | Reduced-motion preference; defaults to the OS `prefers-reduced-motion` setting until the player toggles it |

Changing a key's meaning needs a migration or a new key name.

## Conventions

- **Single file.** Do not split the game into modules or add a build step unless the user asks for it.
- **Match the surrounding style.** Two-space indentation, single quotes in JS, `const` by default, short helper functions, and terse inline comments only where the logic is not obvious.
- **Tuning lives in tables.** Prefer editing `DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, or `THEMES` over hard-coding numbers in logic.
- **Respect accessibility hooks.** New visual effects should go through `screenShake`, `flashHit`/`flashBomb`, or particle helpers so the reduced-motion and quality settings still apply. New UI text should be announced through `announce()` when it changes state.
- **Keep the canvas logical size.** The game renders at `BASE_W` x `BASE_H` (480 x 720) and `resize()` scales it. Coordinates in game logic should stay in that space.
- **Mobile first-class.** Touch controls are shown only on touch devices (`isTouchDevice`). HUD and overlay sizes use `clamp()`, so check narrow widths and the touch layout when changing them.

## Gameplay reference

- Stages: 18 kills (`KILLS_PER_STAGE`) trigger the stage's boss. Defeating the boss advances `stage`. The boss design is `BOSSES[(stage - 1) % BOSSES.length]`, and the sky theme is `THEMES[(stage - 1) % THEMES.length]`.
- Enemy types unlock by stage through `minStage` (tanks from stage 2).
- Weapons: `player.level` raises the spread and lane count. The laser fires faster than the vulcan.
- Score uses a chain multiplier that resets on `breakChain()`.

## Working in this repo

- Work on the branch the session specifies. Do not push to `main` unless asked.
- There is no CI or test command to run. Say so in summaries rather than claiming tests passed.
- Keep this file current when you change controls, storage keys, stage structure, or the file layout.
