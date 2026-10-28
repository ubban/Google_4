# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a single-page browser game, **Sky Fighter**: a Raiden-style vertical shoot-'em-up rendered on a 2D `<canvas>`. The entire game (markup, CSS, and JavaScript) lives in one file, `index.html` (~1,670 lines). `README.md` is a placeholder and has no build or run instructions.

There is no build step, package manager, test suite, or linter. To run the game, open `index.html` in a browser.

## Repository layout

```
index.html   # The whole game: <style> (lines ~10-360), <body> markup (~361-421), <script> (~422-1669)
README.md    # Placeholder
```

## Architecture of `index.html`

### Markup and styling
- `<style>` holds all CSS. Theme and HUD styling, the overlay, touch controls, and the `reduced-motion` body class are defined here.
- `<body>` holds the `<canvas id="gameCanvas">`, the HUD (score, best, multiplier, weapon, hearts, bombs, boss bar), the start/game-over `overlay`, the touch controls (`#touchControls`, joystick, fire and bomb buttons), and a hidden `srLive` region for screen-reader announcements.
- Every element the script uses is looked up by `id` at the top of the IIFE. Renaming an `id` in the markup breaks the game silently at runtime, so update both sides together.

### Script
The script is one IIFE with `'use strict'`. Nothing is exposed on `window`. Its sections, in file order:

1. **DOM references and constants**: element lookups, `STORAGE_*` keys, `announce()`.
2. **Adaptive performance**: `trackPerf()` samples frame time and steps a `quality` level (0 full, 1 medium, 2 low) up or down.
3. **Tuning tables**: `DIFFICULTIES` (spawn rate, enemy speed, shot chance, lives, bombs, boss HP multiplier), `KILLS_PER_STAGE`, `THEMES` (background palettes per stage), `ENEMY_TYPES` (grunt, interceptor, tank), and `BOSSES` (six bosses with colors and HP multipliers).
4. **Persistence and settings**: best score, mute, difficulty, and reduced-motion, all stored in `localStorage` under `skyfighter.*` keys.
5. **Audio**: `ensureAudio()` lazily creates the `AudioContext`. `beep()` and `noiseBurst()` synthesize sound effects; the `sfx` object exposes named effects.
6. **HUD and feedback**: `renderHearts`, `renderBombs`, `flashHit`, `showComboPop`, `showStageBanner`, `screenShake`.
7. **Canvas sizing**: `BASE_W × BASE_H` is 480 × 720. `resize()` scales the canvas to fit the window, capped at 1.6×.
8. **Input**: keyboard, touch joystick, fire/bomb buttons, and mouse. Input is stored in the `keys` map and the `firing`, `joyVec`, and `mouseActive` flags.
9. **Game state and logic**: `resetGame`, `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`, `addScore`, `breakChain`, `useBomb`, `fireWeapon`, `update(dt)`, `applyPowerUp`, `killBoss`, `loseLife`, `endGame`, `togglePause`.
10. **Boss patterns**: `BOSS_PATTERNS` holds one list of bullet-pattern functions per boss design (`fireRadial`, `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`, `fireCross`, `fireHomingSeeded`, `fireDoubleRing`). `updateBoss(dt)` looks up `BOSS_PATTERNS[designIndex]` and cycles through it by `patternIndex`.
11. **Rendering**: `drawClouds`, `drawPlane`, `drawBossShip`, `draw()`.
12. **Main loop**: `loop(now)` clamps `dt` to 50 ms, calls `trackPerf`, then runs `update` and `draw` only while `running && !paused`. It then calls `requestAnimationFrame(loop)`.

### Controls
| Action | Keyboard | Touch / pointer |
|---|---|---|
| Move | Arrow keys or WASD | Left-side joystick, or mouse over canvas |
| Fire | Space (hold) | Fire button (hold), or mouse button on canvas |
| Bomb | X or Shift | Bomb button |
| Pause | P or Esc | Pause button |

Touch controls are shown only when `isTouchDevice` is true.

### Persistence keys
| Key | Value |
|---|---|
| `skyfighter.bestScore` | Number |
| `skyfighter.muted` | `'1'` or `'0'` |
| `skyfighter.difficulty` | `easy`, `normal`, or `hard` |
| `skyfighter.reducedMotion` | `'1'` or `'0'`; defaults to the OS `prefers-reduced-motion` setting until the user toggles it |

Changing a key's name or value format will orphan existing players' saved data.

## Conventions

- **Keep it one file.** The project is deliberately dependency-free. Don't add a bundler, framework, or external script unless the user asks for one.
- **Match the existing style.** The code uses `const`/`let`, arrow functions for callbacks, two-space indentation, single quotes, and short `// ---------- Section ----------` banners to separate sections. Follow these when adding code.
- **Put tunable numbers in the tables.** Difficulty, enemy stats, boss stats, and theme colors live in the constant tables near the top of the script. Add new tunables there, not as inline literals in `update()` or `draw()`.
- **Respect the frame loop.** `update(dt)` and `draw()` run every frame. Avoid allocating large arrays or creating `AudioContext` nodes there without need, and remember that `quality` already throttles visual effects.
- **Respect reduced motion.** Effects that shake or flash should check `reducedMotion` (as `screenShake` and `flashHit` do).
- **Keep accessibility hooks working.** Announcements go through `announce()`, and the overlay and HUD should remain readable with the screen-reader region.

## Testing and verification

No automated tests exist. Verify changes by:

1. Opening `index.html` directly in a browser (or serving the folder with any static server).
2. Playing through at least one full stage and one boss, on each difficulty you touched.
3. Checking the desktop controls (keyboard and mouse) and, when relevant, the touch layout at phone width.
4. Confirming that pause, mute, difficulty, and reduced-motion settings persist across a reload.
5. Watching the browser console for errors. The script runs under `'use strict'`, so typos in variable names fail loudly at runtime and not at load time.

If you add a test harness later, document how to run it here.

## Git workflow

- Develop on the branch the session specifies (currently `claude/gifted-tesla-8j2xx8`). Don't push directly to `main`.
- Commit with clear, descriptive messages. Push with `git push -u origin <branch>`.
- Don't open a pull request unless the user asks for one.

## Known gaps

- `README.md` has no description, setup, or controls information. Expand it if the project is shared more widely.
- No license, tests, or CI are configured.
