# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up that runs in the browser. The entire game is a single self-contained file, `index.html`, with inline CSS and an inline `<script>`. There is no build step, no bundler, no package manager, no external script or stylesheet loads, and no test suite.

Repository layout:

- `index.html` — the whole game (markup, styles, and game logic).
- `README.md` — minimal project readme.
- `CLAUDE.md` — this file.

## Running the game

Open `index.html` directly in a browser, or serve the directory with any static server, for example:

```sh
python3 -m http.server 8000   # then visit http://localhost:8000
```

There is nothing to install or compile.

## Architecture (inside `index.html`)

The script is a single `'use strict'` IIFE (`(function () { ... })();`) containing all `const`/`let` declarations and functions, so nothing is exposed on `window`. Key sections, roughly in file order:

- **Config constants** (around lines 463–515): `STORAGE_*` localStorage keys, `DIFFICULTIES` (spawn rates, enemy speed, shoot chance, starting lives/bombs, boss HP multiplier), `KILLS_PER_STAGE`, `THEMES` (sky gradients and per-stage palettes), `ENEMY_TYPES` (grunt, interceptor, tank), and `BOSSES` (six stage bosses, selected by `(stage - 1) % BOSSES.length`, with `hpMul`).
- **Audio** (`ensureAudio`, `beep`, `noiseBurst`, `sfx`): Web Audio synthesis only; no audio files. Every sound goes through `beep` or `noiseBurst`, which return early when `muted` is set.
- **Adaptive performance** (`trackPerf`, `quality` 0/1/2): frame times are sampled and the render quality is lowered on slow devices.
- **UI helpers** (`renderHearts`, `renderBombs`, `flashHit`, `showComboPop`, `showStageBanner`, `screenShake`, `announce`): HUD and feedback. `announce` feeds an accessibility live region.
- **Input** (around lines 769–870): keyboard, on-screen joystick and buttons for touch, and mouse steering on the canvas.
- **Game state and loop**: `resetGame`, `update(dt)`, `draw()`, `loop(now)` driven by `requestAnimationFrame`. `update` handles movement, firing, collisions, scoring, and stage progression; `draw` renders everything to a single `<canvas id="gameCanvas">`.
- **Weapons and power-ups**: `fireWeapon` (`vulcan` and `laser`), `applyPowerUp` (`life`, `bomb`, `weapon`), `useBomb`.
- **Bosses**: `spawnBoss`, `updateBoss`, `killBoss`, and the `BOSS_PATTERNS` array of bullet-pattern functions (`fireRadial`, `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`, `fireCross`, `fireHomingSeeded`, `fireDoubleRing`).
- **Lifecycle**: `loseLife`, `endGame`, `togglePause`, and the start button handler.

### Controls

- Mouse: move to steer, click to fire.
- Arrows / WASD: move. Space: fire.
- X or Shift: bomb. P or Esc: pause.
- Touch: drag the left zone to fly, tap FIRE, tap BOMB.

### Persisted state

Stored in `localStorage` under the `skyfighter.*` prefix: best score (`bestScore`), mute (`muted`), difficulty (`difficulty`), and reduced motion (`reducedMotion`). Every access should tolerate storage being unavailable (private mode, blocked storage); the current code reads these directly, so keep new reads defensive.

## Conventions

- **Keep it a single file** unless the user asks for a split. Do not introduce a build tool, framework, or npm dependency without explicit approval.
- **Match the existing style**: two-space indentation, single quotes in JS, `const` by default, `camelCase` for functions and variables, `UPPER_SNAKE_CASE` for config constants.
- **Frame-rate independence**: game logic takes a `dt` argument in `update(dt)`. New movement, timers, or cooldowns should scale with `dt` rather than counting frames.
- **Responsive and touch-first**: the canvas is scaled to the viewport (`BASE_W` × `BASE_H` = 480 × 720 logical pixels, see `resize`). Keep changes working at phone width with `touch-action: none` semantics intact.
- **Respect reduced motion and mute**: new visual effects should consult `reducedMotion` where they shake or flash heavily, and new sounds should go through the existing `sfx`/`beep` helpers so mute applies. Announce important game events through `announce()` for screen readers.
- **Keep balance data in the config tables** (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `THEMES`) rather than hardcoding magic numbers in update/draw logic.

## Verification

There is no automated test suite or linter configured. To verify a change:

1. Open `index.html` in a browser (or use the Playwright browser pre-installed in cloud sessions) and play through at least one stage.
2. Check the console for errors, especially during boss fights and game over.
3. Exercise the area you changed on all three difficulties, and on a narrow (phone-width) viewport if the change touches layout, input, or the HUD.

When reporting results, state what was actually run and observed; if a check was not performed, say so.

## Git

- Develop on the branch specified for the session (currently `claude/gifted-tesla-snh1at`). Do not push to other branches without explicit permission.
- Do not open pull requests unless the user asks for one.
- Commit messages should be short and imperative, describing the change (for example, `Add stage-clear bonus score`).
