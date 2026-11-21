# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a small static web project. Its only real content is **Sky Fighter**, a Raiden-style vertical shoot-em-up written as a single self-contained HTML5 canvas page.

| File | Purpose |
| --- | --- |
| `index.html` | The whole game: markup, CSS (`<style>`, ~lines 10–362), and JavaScript (`<script>`, ~line 422 onward), all inline. |
| `README.md` | Placeholder project title. |
| `CLAUDE.md` | This file. |

There is no package manager, build step, bundler, test suite, or linter. Do not add one unless asked.

## Running the game

Open `index.html` directly in a browser, or serve the directory with any static server, for example:

```bash
python3 -m http.server 8000   # then visit http://localhost:8000/index.html
```

Verifying a change means loading the page in a browser and playing it. There are no automated tests to run.

## Architecture of `index.html`

The script is one IIFE (`(function () { 'use strict'; ... })();`). Everything is closure-scoped; nothing is exposed on `window`. Keep new code inside that IIFE.

Major sections, in file order:

1. **DOM references**: `const xxxEl = document.getElementById(...)`, collected at the top.
2. **Persistence keys**: `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion` (all in `localStorage`).
3. **Adaptive quality**: `trackPerf(dt)` samples frame times and steps `quality` between 0 (full), 1 (medium), and 2 (low).
4. **Config tables**:
   - `DIFFICULTIES` (`easy` / `normal` / `hard`): spawn rates, enemy speed, shoot chance, lives, bombs, boss HP multiplier.
   - `THEMES`: six sky palettes cycled per stage.
   - `ENEMY_TYPES`: `grunt`, `interceptor`, `tank`, with `minStage` gating.
   - `BOSSES`: six boss designs, cycled by stage. Attack patterns are in `BOSS_PATTERNS`.
   - `KILLS_PER_STAGE = 18`.
5. **Audio**: a lazily created `AudioContext` (`ensureAudio`) with synthesized `beep` and `noiseBurst` effects. No audio files.
6. **Input**: keyboard (`keys` map), mouse/pointer (`canvasPointFromEvent`), and touch joystick plus on-screen FIRE and BOMB buttons.
7. **Layout**: `BASE_W = 480`, `BASE_H = 720`. `resize()` scales the canvas to fit the viewport.
8. **Game state and logic**: `resetGame`, `update(dt)`, weapons (`fireWeapon`, 3 power levels), bombs (`useBomb`), power-ups (`applyPowerUp`), the kill-chain combo (`addScore`, `breakChain`), boss fights (`updateBoss`, `killBoss`), and `loseLife` / `endGame`.
9. **Rendering**: `draw()` and helpers (`drawPlane`, `drawBossShip`, `drawClouds`). All graphics are drawn with canvas primitives; there are no image assets.
10. **Main loop**: `loop(now)` clamps `dt` to 50 ms, updates and draws only when running and not paused, then calls `requestAnimationFrame`.

Game states are driven by `running`, `paused`, and the `overlay` element. `togglePause()` handles P / Escape.

## Conventions

- **Single file.** Add new HTML, CSS, and JS to `index.html` unless the user asks for a split. Do not introduce a framework, a build step, or external dependencies.
- **No external resources.** The page currently loads nothing from a CDN or network. Keep it that way. Audio is synthesized and graphics are drawn in code.
- **Match the existing style.** Use `const` and `let`, arrow functions for callbacks, 2-space indentation, and single quotes for JS strings. Keep the section banners (`// ---------- Name ----------`).
- **Tune through the config tables.** Balance changes (speed, spawn rates, HP, scores) should go in `DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, or the `BOSS_PATTERNS` constants rather than being hard-coded in logic.
- **Respect accessibility settings.** Anything that shakes the screen, flashes, or animates heavily must honor `reducedMotion` (`body.reduced-motion`). Keep screen-reader announcements going through `announce()`.
- **Respect the performance tier.** New per-frame effects should degrade with `quality` (fewer particles or simpler drawing at higher tiers).
- **Persist only through the existing keys.** If you add saved state, use the `skyfighter.*` prefix, and wrap `localStorage` reads and writes so they still work when storage is unavailable.
- **Keep input parity.** Every action needs a keyboard binding and a touch or on-screen equivalent when it applies to play. The controls hint in the overlay (`#controlsHint`) must stay accurate when controls change.

## Making changes

- Read the relevant section of `index.html` before editing. The file is about 1,670 lines, so search with `grep -n` rather than reading all of it.
- After any gameplay or rendering change, open the page in a browser and check: start, play, pause, lose a life, bomb, boss fight, game over, and restart. Also check a narrow (mobile) viewport.
- Keep commits focused. Commit messages should say what changed and why.
- Do not create a pull request unless the user asks for one.

## Git

- Development branch for this work: `claude/gifted-tesla-j1dre8`.
- Push with `git push -u origin <branch>`. Do not push to other branches without permission.
- Commit only when asked to.
