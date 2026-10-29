# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a small static web project. Its only real content is **Sky Fighter**, a Raiden-style vertical shoot-'em-up written as a single self-contained HTML5 canvas page.

| File | Purpose |
| --- | --- |
| `index.html` | The entire game: markup, CSS (`<style>`), and JavaScript (`<script>`) in one file (~1,670 lines). |
| `README.md` | Placeholder title only. |

There is no package manager, build step, bundler, test suite, linter, or CI configuration. Do not add one unless asked.

## Running the game

No build is needed. Open `index.html` directly in a browser, or serve the directory with any static server, for example:

```bash
python3 -m http.server 8000   # then visit http://localhost:8000/index.html
```

## Architecture of `index.html`

The game script is one IIFE (`(function () { 'use strict'; ... })();`) with no globals. Keep new code inside it.

Major sections, in file order:

- **DOM references**: `const` lookups for every HUD, overlay, and touch element by `id`. Element IDs are load-bearing; renaming one breaks the script.
- **Persistence keys**: `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion` in `localStorage`.
- **Tuning tables**:
  - `DIFFICULTIES` (`easy` / `normal` / `hard`): spawn rate, enemy speed, enemy fire chance, starting lives and bombs, boss HP multiplier.
  - `KILLS_PER_STAGE` (18): kills needed to advance a stage.
  - `THEMES`: six sky palettes that cycle by stage.
  - `ENEMY_TYPES`: `grunt`, `interceptor` (weaving), `tank` (4 HP, unlocks stage 2).
  - `BOSSES`: six bosses, one per stage, each with a color scheme and `hpMul`.
  - `BOSS_PATTERNS` (near line 1372): bullet-pattern functions such as radial, aimed spread, columns, spiral, cross, homing, and double ring.
- **Adaptive quality**: `trackPerf` samples frame time and steps `quality` between 0 (full), 1 (medium), and 2 (low) to keep frame rate up.
- **Audio**: a lazily created `AudioContext` (`ensureAudio`) with `beep` and `noiseBurst` helpers and a `sfx` object. Audio starts on the first user gesture.
- **Input**: keyboard (`keys` map keyed by `e.code`), mouse/pointer steering and fire, and an on-screen joystick plus FIRE and BOMB buttons on touch devices.
- **Game state and loop**: `resetGame`, `update(dt)`, and `draw()`, driven by `requestAnimationFrame` through `loop(now)`. `dt` is clamped to 50 ms.
- **Power-ups**: `maybeSpawnPowerUp` creates `life`, `bomb`, or `weapon` pickups; `applyPowerUp` applies them. Weapon pickups raise the level to 3, then switch between `vulcan` and `laser`.
- **Boss fight**: `spawnBoss`, `updateBoss`, `killBoss`, with a boss health bar.

### Controls (from the in-game hint)

- Move: arrow keys or WASD, or mouse pointer
- Fire: Space, or hold the mouse button
- Bomb: X or Shift
- Pause: P or Esc
- Touch: drag left side to fly, tap FIRE, tap BOMB

## Conventions

- **Keep it one file.** The project is deliberately a single HTML file with no dependencies. Do not split out modules, add a framework, or add a bundler without an explicit request.
- **Match the existing style.** Two-space indentation, single quotes in JavaScript, `const`/`let` (no `var`), arrow functions for callbacks, and `// ----- Section -----` comment banners between major blocks.
- **Tuning belongs in the tables.** Change balance numbers in `DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, or `THEMES` rather than hard-coding them in logic.
- **Respect the game's accessibility features.** Reduced motion (`skyfighter.reducedMotion`, which defaults to the `prefers-reduced-motion` media query) is read by `screenShake()`, which scales magnitude to 15% and caps duration at 0.1 s, and it toggles the `reduced-motion` body class. `announce()` feeds the `aria-live` region. New visual effects should check `reducedMotion`, and new state changes should call `announce` where appropriate.
- **Persist only through the existing keys.** If you add a new saved setting, follow the `skyfighter.*` naming and wrap nothing in extra abstractions.
- **Keep the canvas base size.** Game logic is laid out for `BASE_W = 480`, `BASE_H = 720`; `resize()` scales the canvas to fit the window.

## Verifying changes

There is no automated test suite. After editing `index.html`:

1. Open it in a browser (desktop and, if touched, a mobile viewport) and check the console for errors.
2. Play a run through at least one stage transition and one boss, and confirm HUD values (score, best, lives, bombs, weapon level) update.
3. Check pause (P/Esc), mute, difficulty switching, and reduced motion.
4. For touch changes, test the joystick and FIRE/BOMB buttons with touch emulation.

Report what you actually ran. If you could not run the game, say so rather than claiming it works.

## Git and branches

- Develop on the branch you were assigned. Do not push to `main` unless asked.
- Write clear, descriptive commit messages in the imperative mood, with a short subject line and a body explaining the why when it is not obvious.
- Do not create a pull request unless the user asks for one.
- Do not commit generated files or editor/OS artifacts. The repository currently tracks only `README.md` and `index.html`.
