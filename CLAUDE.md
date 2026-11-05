# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

**Sky Fighter** is a single-file, Raiden-style vertical shoot-'em-up that runs in the browser on a `<canvas>`. There is no build step, no package manager, and no test suite.

Files:

- `index.html` — the entire game: inline CSS (`<style>`), markup (HUD, overlays, touch controls), and one inline `<script>` IIFE containing all game logic, rendering, audio, and input.
- `README.md` — one-line project title. Keep it short.

To run the game, open `index.html` directly in a browser (or serve the directory with any static server). Nothing needs to be installed.

## Architecture (inside `index.html`)

The script is wrapped in `(function () { 'use strict'; ... })()`. Everything is private to that closure.

- **DOM references** are grabbed once at the top of the script (`canvas`, `overlay`, HUD elements, touch controls, etc.). Look up new elements the same way rather than querying repeatedly.
- **Persistence** uses `localStorage` with keys prefixed `skyfighter.`:
  - `skyfighter.bestScore`
  - `skyfighter.muted`
  - `skyfighter.difficulty`
  - `skyfighter.reducedMotion`
- **Config tables** define the tuning data and are the first place to change balance:
  - `DIFFICULTIES` — `easy` / `normal` / `hard`: spawn timing, enemy speed, enemy fire chance, starting lives and bombs, boss HP multiplier.
  - `KILLS_PER_STAGE` — kills needed to advance a stage (currently 18).
  - `THEMES` — six sky palettes, cycled per stage.
  - `ENEMY_TYPES` — `grunt`, `interceptor` (weaving), `tank` (4 HP). Each has a `minStage` gate.
  - `BOSSES` — six bosses (VANGUARD → TITAN), each with colors and an `hpMul`.
  - `BOSS_PATTERNS` — bullet patterns (radial, aimed spread, columns, spiral, cross, homing, double ring).
- **Main loop**: `requestAnimationFrame(loop)` → `update(dt)` → `draw()`. `dt` (seconds) is capped at 0.05 in `loop()`; keep all movement and timers `dt`-based, not frame-based.
- **Adaptive quality**: `trackPerf(dt)` watches a rolling 45-frame average and steps `quality` between full / medium / low. Rendering code should respect `quality` and `reducedMotion` when adding expensive effects.
- **Audio** is synthesized with the Web Audio API (`ensureAudio`, `beep`, `noiseBurst`, the `sfx` object). Audio is created lazily on the first user gesture; call `ensureAudio()` from any new input handler that plays sound.
- **Accessibility**: `announce(text)` writes to the `#srLive` live region. Use it for important state changes (stage clear, bomb use, game over).
- **Responsive canvas**: `BASE_W = 480`, `BASE_H = 720`. `resize()` scales the canvas to fit the viewport; game coordinates stay in base units.

### Game systems

- **Player**: 3 lives and 2 bombs on normal (varies by difficulty). Hit → `loseLife()`. Bombs → `useBomb()`, which clears the screen.
- **Weapons**: two types, `vulcan` and `laser`, each with levels 1–3 controlled in `fireWeapon()`. A weapon power-up raises the level; past level 3 it toggles the weapon type and resets to level 1 (`applyPowerUp()`).
- **Power-ups** (`maybeSpawnPowerUp()`): `life`, `bomb`, `weapon`, dropped by destroyed enemies at fixed probabilities.
- **Scoring**: `addScore()` increments a chain counter; the multiplier is `1 + min(9, floor(chain / 5))`, so it tops out at ×10. `breakChain()` resets it. Best score is written to `localStorage` on game over.
- **Game state**: `resetGame()` initializes state; `endGame()` shows the overlay with final score, best, and stage; `togglePause()` handles pause.

## Controls

- **Keyboard**: Arrows / WASD to move, Space to fire (hold for auto-fire), X or Shift for bomb, P or Esc to pause.
- **Mouse**: move to steer, hold click to fire.
- **Touch**: drag on the left to fly (virtual joystick), tap FIRE, tap BOMB.

Input handlers live in the `// ---------- Input ----------` block. Keyboard state is tracked in the `keys` object keyed by `e.code`; pointer and touch fire state is the `firing` flag.

## Conventions

- **Keep it one file.** Do not split into modules, add a bundler, or introduce a framework unless the user asks. The zero-setup property is intentional.
- **No external dependencies.** Do not add CDN scripts or fonts. Use the system font stack already declared on `body`.
- **Match existing style**: 2-space indentation, single quotes in JS, `const`/`let` (no `var`), arrow functions for callbacks, small helper functions above their first use.
- **Balance changes** belong in the config tables (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `THEMES`) rather than hard-coded in logic.
- **Mobile first**: any new control must work with touch, keep the `touch-action: none` behavior intact, and not block the canvas.
- **Respect the user's motion preference**: effects that shake or flash should check `reducedMotion`.

## Verifying changes

There are no automated tests or linters configured. To verify a change:

1. Open `index.html` in a browser and check the browser console for errors.
2. Play through at least one stage on each affected difficulty, including a boss fight if boss or bullet code changed.
3. Check the narrow (phone) layout and touch controls if HUD, overlay, or input code changed.
4. If you have Node available, a quick syntax check of the inline script is useful, e.g. extract the `<script>` block and run `node --check` on it.

## Git workflow

- Default branch work: develop on the branch the task specifies (currently `claude/gifted-tesla-1qp7or` for this repo's assigned feature work).
- Commit with clear, descriptive messages. Push with `git push -u origin <branch>`.
- Do not open pull requests unless explicitly asked.
