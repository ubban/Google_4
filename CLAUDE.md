# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a small static web project. Its only game is **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs entirely in the browser as a single HTML5 canvas page.

- No build step, bundler, package manager, or test suite.
- No external runtime dependencies. All CSS and JavaScript are inline in `index.html`.
- Persistence is limited to `localStorage`.

## Repository layout

| Path         | Purpose                                                                 |
|--------------|-------------------------------------------------------------------------|
| `index.html` | The entire game: markup (~lines 1–420), inline `<style>`, and inline `<script>` (~line 422 onward). |
| `README.md`  | Minimal project readme.                                                 |
| `CLAUDE.md`  | This file.                                                              |

To run the game, open `index.html` in a browser. There is nothing to install or compile.

## Architecture of `index.html`

**Markup.** A `#container` holds the `<canvas id="gameCanvas">`, a `#hud` (score, best, multiplier, weapon, hearts, bombs), a boss health bar (`#bossBarWrap`), combo and stage popups, a `#topBar` (mute, reduced-motion, pause buttons), touch controls (`#touchControls` with a joystick, FIRE and BOMB), and an `#overlay` start/game-over card with difficulty buttons (`.diffBtn`).

**Script.** Everything lives in one IIFE-style block. Key sections, roughly in source order:

1. **DOM references.** `const xxxEl = document.getElementById(...)` for every element the script touches.
2. **Storage keys.** `STORAGE_BEST`, `STORAGE_MUTE`, `STORAGE_DIFF`, `STORAGE_REDUCED` (prefixed `skyfighter.`).
3. **Accessibility and perf helpers.** `announce()` writes to the `#srLive` region. `trackPerf()` samples frame times and sets `quality` (0 full, 1 medium, 2 low) to reduce effects on slow devices.
4. **Tuning tables.**
   - `DIFFICULTIES`: `easy`, `normal`, `hard`. Each sets spawn rates, enemy speed, shoot chance, starting lives and bombs, and a boss HP multiplier.
   - `KILLS_PER_STAGE` (18): kills needed to advance a stage.
   - `THEMES`: six sky palettes, cycled per stage.
   - `ENEMY_TYPES`: `grunt`, `interceptor` (weaving), `tank` (4 HP, unlocks at stage 2).
   - `BOSSES`: six bosses (VANGUARD → TITAN), each with its own colors and `hpMul`.
5. **Audio.** A lazily created `AudioContext` (`ensureAudio()`). Sound effects are synthesized with `beep()` and `noiseBurst()`, mapped through the `sfx` object. Nothing is loaded from disk.
6. **Input.** Keyboard (`keys`), mouse (`mouseTarget` steers the ship, click fires), and touch (`joystickStart/Move/End`, `fireBtn`, `bombBtn`). `firing` is the shared fire state.
7. **Game state.** Module-level `let` variables: `player`, `bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`, `boss`, `score`, `lives`, `bombCount`, `stage`, `kills`, `chain`/`multiplier`, and so on.
8. **Game loop.** `requestAnimationFrame`-driven update and draw. Work is split across spawning, player and enemy updates, collision, power-up pickup, boss logic, particles, and HUD refresh.

**Game mechanics to preserve when editing:**

- Weapons are Vulcan and Laser, each with 3 power levels, gained from power-ups.
- Smart bombs clear the screen and damage bosses. The count is limited per difficulty.
- Stages advance after `KILLS_PER_STAGE` kills and end in a boss fight. The boss pattern and HP bar are tied to `BOSSES[]`.
- A kill chain raises the score multiplier (`chain`, `chainTimer`, `multiplier`) and shows combo popups.
- Power-ups include weapon level, bomb, and extra life.

## Persistent state (`localStorage`)

| Key                         | Value                    | Purpose                           |
|-----------------------------|--------------------------|-----------------------------------|
| `skyfighter.bestScore`      | number                   | Best score across sessions        |
| `skyfighter.muted`          | `'1'` / `'0'`            | Sound mute                        |
| `skyfighter.difficulty`     | `easy` / `normal` / `hard` | Selected difficulty             |
| `skyfighter.reducedMotion`  | `'1'` / `'0'`            | Reduced motion (screen shake, flash, pulse). Defaults to the `prefers-reduced-motion` media query when unset. |

Any new persisted setting should follow the `skyfighter.` prefix convention and be read with a fallback so a missing or malformed value does not break the game.

## Conventions

- **Keep it a single file.** Add new CSS to the `<style>` block and new JavaScript to the existing `<script>` block unless there is a clear reason to split. Do not introduce a build step or external dependencies without asking.
- **Canvas is fixed-size.** Game logic uses a base resolution of `BASE_W × BASE_H` (480 × 720). `resize()` scales the canvas to fit the window (max 1.6×). Coordinates in game logic should stay in base units.
- **Respect the three input paths.** Any control change should work with keyboard, mouse, and touch. The on-screen touch controls are hidden on non-touch devices (`isTouchDevice`).
- **Respect reduced motion.** New screen shake, flashes, or pulsing effects should check `reducedMotion` and `quality` before running.
- **Route user-visible status through `announce()`** when it matters to screen-reader users (score changes, stage changes, pause, game over).
- **Keep the HUD in sync.** Update the relevant `*El` elements when state changes, and keep the `aria-label`/`title` text on icon buttons accurate.
- **Match existing style.** Two-space indentation, single quotes in JavaScript, `const` by default, and descriptive names like `spawnEnemy` or `flashHit`.

## Testing and verification

There is no automated test suite or linter. Verify changes by:

1. Opening `index.html` in a desktop browser and playing through at least one stage transition and a boss fight.
2. Checking all three difficulties if you touched `DIFFICULTIES` or spawn logic.
3. Checking the touch layout (browser device emulation) if you touched controls or the HUD.
4. Confirming the mute, reduced-motion, and pause buttons still toggle and persist across reloads.
5. Watching the browser console for errors. The game should run without any.

## Git workflow

- Default branch: `main`. Feature work for this task is developed on `claude/gifted-tesla-alexdf`.
- Write clear commit messages that describe the gameplay or UI change, not just the file touched.
- Do not open a pull request unless the user asks for one.
