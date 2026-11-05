# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a small static web project. Its only real content is **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs in a single HTML file with no build step, no package manager, and no dependencies.

| File | Purpose |
| --- | --- |
| `index.html` | The entire game: markup (~lines 1–420), inline CSS (lines 10–359), and an inline IIFE-wrapped `<script>` (lines 422–1667). |
| `README.md` | Placeholder title only. |
| `CLAUDE.md` | This file. |

There is no test suite, linter, or CI configuration in the repo.

## Running the game

No install step is needed. Open `index.html` directly in a browser, or serve the directory with any static server, for example:

```bash
python3 -m http.server 8000   # then visit http://localhost:8000/
```

## Architecture of `index.html`

The file is organized in three sections.

1. **Markup.** A `#container` holds `<canvas id="gameCanvas">`, the HUD (`#hud`, `#topBar`, score/best/multiplier, weapon, hearts, bombs, boss bar, combo and stage banners), touch controls (`#touchControls`, with a joystick and FIRE/BOMB buttons), and the `#overlay` start/pause/game-over card with difficulty buttons (`.diffBtn`) and `#startBtn`.
2. **CSS.** Inline `<style>`. Layout and effects (vignette, flash, shake classes) live here.
3. **Script.** One `(function () { 'use strict'; ... })();` IIFE. Nothing is exported to `window`, so keep new code inside this scope.

Inside the script, the code is grouped by concern, with comment banners such as `// ---------- Adaptive performance ----------`:

- **Config constants:** `DIFFICULTIES` (easy/normal/hard: spawn rates, enemy speed, shoot chance, lives, bombs, boss HP multiplier), `KILLS_PER_STAGE` (18), `THEMES` (six sky palettes, indexed by `(stage - 1) % 6`), `ENEMY_TYPES` (`grunt`, `interceptor`, `tank`), and `BOSSES` (six bosses, cycled by stage, each with colors and an `hpMul`).
- **Persistence:** `localStorage` keys prefixed `skyfighter.` (`bestScore`, `muted`, `difficulty`, `reducedMotion`). All reads and writes go through these constants. Wrap any new storage access in a try/catch, because storage can be unavailable (private mode, blocked site data).
- **Adaptive quality:** `trackPerf` averages the last 45 frame times and steps `quality` between 0 (full), 1 (medium), and 2 (low) to hold roughly 40–56 FPS.
- **Audio:** Web Audio API. `ensureAudio()` lazily creates the context on first user gesture. `beep()` and `noiseBurst()` are the primitives, and the `sfx` object exposes named sounds. Respect the `muted` flag.
- **Input:** keyboard (`keys[e.code]`; arrows/WASD move, Space fires, X or Shift bombs, P or Esc pauses), mouse on the canvas (steer and click to fire), and touch (a floating joystick on the left and fire/bomb buttons). Touch controls are enabled only when `isTouchDevice` is true.
- **Game state and loop:** `resetGame()` initializes the `player`, `enemies`, `bullets`, `powerUps`, `explosions`, and boss state. `update(dt)` advances the simulation, `draw()` renders, and `loop(now)` drives both through `requestAnimationFrame`. `dt` is clamped to 50 ms. `update` runs only when `running && !paused`.
- **Scoring:** `addScore()` applies the combo multiplier. `breakChain()` resets it. `endGame()` writes `bestScore` when beaten.
- **Power-ups:** dropped from `maybeSpawnPowerUp`, with types `life`, `bomb`, and `weapon`. `applyPowerUp()` handles pickup. The weapon toggles between `vulcan` and `laser`.
- **Bosses:** `spawnBoss()` and `BOSS_PATTERNS` (an array of bullet-pattern functions such as `fireRadial`, `fireSpiralBurst`, `fireHomingSeeded`). `updateBoss()` picks patterns; `killBoss()` advances the stage.
- **Rendering:** `drawPlane`, `drawBossShip`, `drawClouds`, and `draw()`. The canvas is a fixed 480×720 logical size (`BASE_W` × `BASE_H`) scaled by CSS in `resize()`.

## Conventions

- **Keep it a single file.** The project is deliberately dependency-free and buildless. Do not add a bundler, framework, or package manager unless the user asks for one.
- **Match the existing style.** Use `const`/`let`, arrow functions for callbacks, two-space indentation, single quotes in JS, and the `// ---------- Section ----------` comment banners.
- **Stay inside the IIFE.** Expose nothing globally. Add config to the existing constant tables rather than hard-coding values in logic.
- **Respect accessibility hooks.** Changes to motion or effects should honor `reducedMotion`, and new announcements should go through `announce()` for the `#srLive` region.
- **Preserve both input paths.** Any new control must work with keyboard, mouse, and touch.
- **Keep the logical resolution.** Gameplay coordinates assume 480×720. Scale via CSS, not by changing `BASE_W`/`BASE_H` casually.

## Making changes

- Read the relevant section of `index.html` before editing. The file is large (~1,670 lines), so search for the banner comment or function name rather than reading all of it.
- Verify changes by loading the page in a browser and playing a few stages. There are no automated tests, so manual verification is the check.
- Check the browser console for errors after any change to the script.
- Commit with clear messages describing the gameplay or code change.
