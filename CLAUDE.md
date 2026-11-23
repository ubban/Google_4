# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a small, dependency-free browser game called **Sky Fighter**: a Raiden-style vertical shoot-'em-up. The entire app is one file, `index.html`, containing inline CSS, markup, and a single IIFE-wrapped `<script>`. `README.md` is a placeholder. There is no build step, package manager, test suite, or linter.

## Repository layout

```
index.html   # The whole game: <style> (~lines 10–400), overlay markup (~400–420), <script> (~422–end)
README.md    # Placeholder
```

## Running the game

No build is needed. Open `index.html` directly in a browser, or serve the directory with any static server (for example `python3 -m http.server`) if a local origin is needed. localStorage is used for persistence, so `file://` works in most browsers but a served origin gives the most consistent behavior.

## Code structure (inside `index.html`)

The script is one `(function () { 'use strict'; ... })();` block. Key sections, in order of appearance:

- **DOM references**: `const` lookups for canvas, HUD, overlay, and touch controls.
- **Storage keys**: `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion`.
- **Performance tracking**: `trackPerf` adjusts a `quality` level (0 full, 1 medium, 2 low) from average frame time.
- **Config tables**: `DIFFICULTIES` (easy/normal/hard), `KILLS_PER_STAGE`, `THEMES` (per-stage sky palettes), `ENEMY_TYPES` (grunt, interceptor, tank), `BOSSES` (per-stage boss designs, cycled by stage).
- **Audio**: lazily created `AudioContext` (`ensureAudio`), with `beep`, `noiseBurst`, and a `sfx` object. Audio starts only after a user gesture.
- **HUD helpers**: hearts, bombs, combo pop, stage banner, screen shake, flash effects.
- **Input**: keyboard (`keys` map), mouse (steer and fire on canvas), and touch (virtual joystick on `moveZone`, `fireBtn`, `bombBtn`).
- **Canvas sizing**: `BASE_W = 480`, `BASE_H = 720`, scaled to fit the window (max 1.6x).
- **Game state**: `resetGame()` initializes `player`, `bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`, and `boss`.
- **Rendering**: `drawPlane`, `drawBossShip`, and the `draw()` pass.
- **Update logic**: `update(dt)`, `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`, `useBomb`, and collision handling.
- **Main loop**: `loop(now)` runs on `requestAnimationFrame`, clamps `dt` to 0.05s, and only updates and draws when `running && !paused`.

## Conventions

- **Keep it a single file.** Do not introduce a bundler, framework, or npm dependencies unless the user asks for that change.
- **Stay in the existing style.** Use `const`/`let`, the `'use strict'` IIFE, camelCase names, and `SCREAMING_SNAKE_CASE` for config constants.
- **Put tunables in the config tables** (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `THEMES`) rather than hard-coding magic numbers in update or draw code.
- **Guard browser APIs.** `localStorage` reads and writes are used directly today; if you add new ones, keep them simple and consistent with the existing keys. Audio must be created only after a user gesture.
- **Respect reduced motion and quality.** New visual effects should honor `reducedMotion` and the `quality` level, as existing particle and shake effects do.
- **Preserve touch support.** Changes to controls must keep mouse, keyboard, and touch paths working. The canvas uses `touch-action: none`.
- **Mind phone widths.** Layout must work at phone size with no horizontal page scroll. The body is `overflow: hidden`, so the canvas scales to fit.

## Verifying changes

There is no automated test suite. To verify a change:

1. Open `index.html` in a browser (or serve it locally) and check the browser console for errors.
2. Start a game, then exercise the path you changed: movement, firing, bombs, pause (`P`/`Esc`), stage transitions (every 18 kills), boss fights, and game over.
3. Check each difficulty (Easy/Normal/Hard) if you touched `DIFFICULTIES` or spawn logic.
4. Check the mute, reduced-motion, and difficulty toggles persist across reloads.
5. For UI or layout changes, check at a phone-sized viewport and a desktop viewport.

If you cannot run a browser in the current environment, say so and verify by reading the code.

## Git

- Develop on the branch the session specifies. Do not push to other branches without explicit permission.
- Do not create pull requests unless the user asks for one.
- Write commit messages that describe the change in the imperative mood.
