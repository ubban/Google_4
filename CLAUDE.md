# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a small static web project. Its only real content is **Sky Fighter**, a
Raiden-style vertical shoot-'em-up that runs entirely in the browser on an HTML5
`<canvas>`.

| File         | Purpose                                                        |
|--------------|----------------------------------------------------------------|
| `index.html` | The whole game: markup (~lines 1–420), CSS (inline `<style>`), and JavaScript (inline `<script>` starting ~line 422). |
| `README.md`  | Minimal placeholder. Contains only a title and a timestamp line. |

There is no build step, no package manager, no bundler, no framework, and no test suite.
Do not add tooling (npm, bundlers, linters, test runners) unless the user asks for it.

## Running the game

Open `index.html` directly in a browser, or serve the directory with any static server, e.g.:

```bash
python3 -m http.server 8000   # then visit http://localhost:8000/index.html
```

There is nothing to install or compile.

## Architecture of `index.html`

The inline script is a single IIFE (`(function () { 'use strict'; ... })();`). Everything
lives inside it, so no globals are exposed. Key sections, in source order:

1. **DOM references**: `document.getElementById(...)` lookups for HUD, overlay, and touch controls.
2. **Persistence keys**: `STORAGE_BEST`, `STORAGE_MUTE`, `STORAGE_DIFF`, `STORAGE_REDUCED`
   (all prefixed `skyfighter.`), stored in `localStorage`.
3. **Adaptive performance**: `trackPerf()` samples frame time and steps a `quality` level
   (0 full, 1 medium, 2 low) up or down.
4. **Tuning data (constants)**:
   - `DIFFICULTIES`: spawn rate, enemy speed and fire chance, lives, bombs, boss HP multiplier.
   - `KILLS_PER_STAGE` (18): kills needed to advance a stage.
   - `THEMES`: sky gradients, cloud and sun colors, and stage names.
   - `ENEMY_TYPES`: `grunt`, `interceptor` (weaving), `tank`. Each has size, HP, speed, score, and `minStage`.
   - `BOSSES`: six bosses (VANGUARD → TITAN). Each has colors and an `hpMul`.
5. **Audio**: a lazily created `AudioContext` (`ensureAudio()`), with `beep()` and
   `noiseBurst()` building synthesized sound effects. There are no audio files.
6. **Input**: keyboard (`keys[e.code]`), a touch joystick and fire/bomb buttons, and mouse
   (move and fire on canvas).
7. **Game state**: `resetGame()` initializes `player`, `bullets`, `enemies`, `enemyBullets`,
   `particles`, `powerUps`, and `boss`.
8. **Simulation**: `update(dt)` advances the game each frame. Helpers include
   `spawnEnemy()`, `spawnBoss()`, `fireWeapon()`, `useBomb()`, `applyPowerUp()`,
   `updateBoss()`, `loseLife()`, and `endGame()`. Boss attack patterns are the
   `fire*` helpers plus the `BOSS_PATTERNS` array.
9. **Rendering**: `draw()` renders everything to the canvas. Helpers include `drawPlane`,
   `drawBossShip`, and `drawClouds`.
10. **Loop**: `loop(now)` runs on `requestAnimationFrame`. `dt` is clamped to 50 ms.
    The game pauses when `paused` is set.

### Conventions used in the code

- Logical play-field size is `BASE_W = 480` × `BASE_H = 720`. `resize()` scales the canvas to fit the viewport.
- `dt` is in seconds. Per-frame movement is multiplied by `dt`, or by a frame-based counter where noted.
- User-visible text updates go through the `announce()` helper, which writes to the
  `srLive` ARIA live region. Keep screen-reader support intact when adding HUD messages.
- Reduced motion is respected. `reducedMotion` comes from `prefers-reduced-motion` or the
  user's saved choice, and toggles the `reduced-motion` body class. Visual effects such as
  screen shake should honor it.
- Mute and difficulty choices persist to `localStorage`. Every storage access is a plain
  call, so keep it that way or wrap it in try/catch if you add more. Private or blocked
  storage can throw.

## Making changes

- **Keep it a single file.** New game logic, styles, and scripts go in `index.html` unless
  the user asks for a split. Do not add build steps or external dependencies.
- **Keep constants as constants.** Balance changes belong in `DIFFICULTIES`, `ENEMY_TYPES`,
  `BOSSES`, `THEMES`, or `KILLS_PER_STAGE`, not in scattered literals.
- **Preserve the adaptive quality path.** Expensive visual effects should check `quality`
  so low-end devices still run smoothly.
- **Test in a browser.** Nothing runs automatically. For gameplay changes, load the page
  and check start, fire, bomb, a stage transition, a boss fight, game over, and the best-score
  update. Also check both desktop keyboard and touch layouts when touching input or CSS.
- **Mobile layout matters.** The touch controls and HUD are part of the design. Verify changes
  at a phone-sized viewport.

## Git and workflow

- Default branch is not the place for direct work. For this session, develop on
  `claude/gifted-tesla-4r9y6u` and push there with `git push -u origin <branch>`.
- Commit messages should be short, imperative, and describe the gameplay or file change.
- Do not commit secrets or local settings. There are none expected in this repo.
- Do not open a pull request unless the user asks for one.

## Known gaps

- `README.md` is a placeholder and does not describe the game. Update it if the project's
  purpose changes.
- There are no automated tests, linters, or CI configuration.
