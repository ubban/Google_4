# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo is a single-file HTML5 canvas game, **Sky Fighter**: a Raiden-style vertical shoot-em-up.

- `index.html` (~1,670 lines) contains all markup, CSS, and JavaScript. There are no external assets, no bundler, and no package manifest.
- `README.md` is a placeholder and has no setup instructions.

To run the game, open `index.html` in a browser. Nothing needs to be installed.

## Repository layout

| Path | Contents |
| --- | --- |
| `index.html` | The whole game: `<style>` (~lines 10–360), HTML body (~361–421), and the inline `<script>` (~422 to end) |
| `README.md` | Placeholder |

Inside the script, the code is organized in this order:

1. **DOM references and storage keys**: `STORAGE_*` constants for `localStorage`.
2. **Tuning tables**: `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES`, `ENEMY_TYPES`, `BOSSES`.
3. **Persistence and settings**: best score, mute, difficulty, reduced motion.
4. **Audio**: a lazily created `AudioContext` with `beep`, `noiseBurst`, and a `sfx` map. Sound starts only after a user gesture (`ensureAudio`).
5. **HUD and effects**: hearts, bombs, combo pop, stage banner, screen shake, hit and bomb flashes.
6. **Layout and input**: `resize()` scales a fixed 480×720 logical canvas. Keyboard, touch joystick, and mouse handlers are set up here.
7. **Game state and logic**: `resetGame`, `spawnEnemy`, `spawnBoss`, `useBomb`, `update(dt)`, `updateBoss(dt)`.
8. **Rendering**: `drawPlane`, `drawBossShip`, `drawClouds`, `draw()`.
9. **Main loop**: `loop(now)` runs on `requestAnimationFrame`. `trackPerf` lowers the `quality` level (0 full, 1 medium, 2 low) when frame times are poor.

## Gameplay summary

- **Stages**: each stage takes `KILLS_PER_STAGE` (18) kills, then a boss fight. Bosses come from the `BOSSES` array, and the background cycles through `THEMES`.
- **Enemies**: `grunt` (stage 1+), `interceptor` (stage 1+, weaves), `tank` (stage 2+, 4 HP).
- **Weapons**: Vulcan and Laser, each with 3 power levels, raised by power-ups.
- **Bombs**: limited smart bombs that clear the screen and damage bosses.
- **Score**: a kill-chain multiplier (`chain`, `chainTimer`, `multiplier`).
- **Difficulty**: `easy`, `normal`, `hard`. These set spawn rate, enemy speed, shot chance, starting lives, starting bombs, and boss HP multiplier.

## Persistence

All state is stored in `localStorage` under the `skyfighter.*` prefix:

| Key | Meaning |
| --- | --- |
| `skyfighter.bestScore` | Best score |
| `skyfighter.muted` | `'1'` when muted |
| `skyfighter.difficulty` | `easy` / `normal` / `hard` |
| `skyfighter.reducedMotion` | `'1'` or `'0'`. If unset, the `prefers-reduced-motion` media query decides the default. |

Renaming a key or changing its meaning will reset that setting for existing players, so treat the keys as a stable interface.

## Input

- **Keyboard**: Arrows or WASD to move, Space to fire, X or Shift to bomb, P or Esc to pause. Space's default is prevented so the page doesn't scroll.
- **Mouse**: move to steer, press and hold to fire (released on window `mouseup`). Coordinates are mapped through `canvasPointFromEvent`, which accounts for the CSS scale.
- **Touch**: a left-side drag zone acts as a virtual joystick (`joystickStart`/`joystickMove`, 50px max radius). FIRE and BOMB are buttons. The touch controls are shown only when `isTouchDevice` is true.

Controls are listed in the on-screen `#controlsHint`. Update it whenever a binding changes.

## Conventions

- **Plain JavaScript, no dependencies.** Don't add libraries, a build step, or a framework unless the user asks. The single-file layout is deliberate.
- **Keep the existing style.** The script uses `const`/`let`, arrow functions in event handlers, 2-space indentation, and short inline comments only where the logic isn't obvious. Match this when editing.
- **Tuning belongs in the tables.** Put balance numbers in `DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, and `THEMES`, not scattered through the update logic.
- **Frame-rate independence.** Update functions take `dt`. Scale movement and timers by it instead of assuming 60 fps.
- **Accessibility.** Respect the reduced-motion setting when adding screen shake, flashes, or pulses. Existing code skips hit/bomb flashes entirely and scales screen shake down to 15% with a 0.1s cap. Announce important state changes through `announce()` (the `#srLive` live region).
- **Audio** only starts after a user gesture. Call `ensureAudio()` from any new input path that plays sound.

## Verifying changes

There is no test suite, linter, or build. Verify this way:

1. **Syntax check the inline script** (Node 18+):
   ```bash
   awk '/<script>/{f=1;next}/<\/script>/{f=0}f' index.html > "$TMPDIR/game.js" && node --check "$TMPDIR/game.js"
   ```
   This prints nothing on success. (`$TMPDIR` falls back to `/tmp` on most systems; set it to any writable scratch directory if needed.)
2. **Play it in a browser.** Start a game on each difficulty, check that bosses spawn at the end of a stage, pause and resume, and check the game-over screen and best-score persistence. On a narrow viewport or with touch emulation, check the touch controls.
3. **Check reduced motion** by toggling the motion button and confirming hit and bomb flashes stop and screen shake is visibly weaker.

## Git workflow

- Develop on the branch the task names. Use descriptive commit messages that explain the gameplay or code change.
- The default branch is not the working branch. Don't push directly to a branch other than the one specified.
