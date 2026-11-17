# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a small, dependency-free browser game, **Sky Fighter**: a Raiden-style vertical shoot-'em-up rendered on an HTML5 `<canvas>`.

The entire game lives in one file, `index.html` (~58 KB), with inline `<style>` and a single inline `<script>` wrapped in an IIFE. `README.md` contains only the title.

There is no build system, package manager, test suite, linter, or CI configuration.

## Repository layout

```
.
├── index.html   # The whole game: markup (#overlay, #hud, #touchControls, <canvas id="gameCanvas">), CSS, and JS
├── README.md    # Title only
└── CLAUDE.md    # This file
```

## Running and testing

- **Run:** open `index.html` directly in a modern browser, or serve the directory with any static server (e.g. `python3 -m http.server`) and visit it.
- **Build:** none required.
- **Tests:** none exist. Verify changes by playing the game in a browser. For automated checks, a Playwright script can load `file://…/index.html` and drive the canvas.

## Code architecture (inside `index.html`)

The script is a single closure. Major sections, in the order they appear:

| Area | Key identifiers | Notes |
|---|---|---|
| Persistence | `STORAGE_BEST`, `STORAGE_MUTE`, `STORAGE_DIFF`, `STORAGE_REDUCED` | All keys are prefixed `skyfighter.` and stored in `localStorage`. |
| Difficulty | `DIFFICULTIES` (`easy` / `normal` / `hard`) | Tunes spawn rates, enemy speed, shoot chance, starting lives and bombs, and boss HP multiplier. |
| Stage data | `KILLS_PER_STAGE`, `THEMES`, `ENEMY_TYPES`, `BOSSES` | Stages advance after a kill quota; each stage ends in a boss. |
| Audio | `ensureAudio()`, `beep()`, `noiseBurst()`, `EXPLODE_SFX` | Web Audio API, synthesized in code (no audio files). Created lazily on first user gesture. |
| Layout | `BASE_W = 480`, `BASE_H = 720`, `resize()` | Fixed logical resolution scaled to fit the viewport. |
| Input | `keys`, `keydown`/`keyup` listeners, `joystickStart/Move/End`, `canvasPointFromEvent` | Keyboard plus on-screen joystick/touch controls, enabled when a touch device is detected. |
| Game state | `resetGame()`, `update(dt)`, `applyPowerUp()`, `fireWeapon()`, `useBomb()` | Core simulation, updated per frame with a clamped `dt`. |
| Bosses | `BOSS_PATTERNS`, `updateBoss()`, `killBoss()` | Each pattern is a `fire*` helper (radial, aimed spread, spiral, cross, homing, etc.). |
| Rendering | `draw()`, `drawPlane()`, `drawBossShip()`, `drawClouds()` | Canvas 2D drawing only. |
| UI feedback | `renderHearts()`, `renderBombs()`, `showComboPop()`, `showStageBanner()`, `screenShake()`, `flashHit()` | DOM-based HUD updates plus canvas effects. |
| Loop | `loop(now)`, `trackPerf(dt)` | `requestAnimationFrame` loop. `trackPerf` adapts performance over time. |

### Controls

- Move: arrow keys or `A`/`D` (and the other arrows), or touch joystick on touch devices.
- Fire: `Space` (hold), or touch fire.
- Bomb: `X` or `Shift`.
- Pause: `P` or `Escape`.
- Mute, difficulty, and reduced-motion toggles are in the UI.

## Conventions

- **Single file.** Keep the game in `index.html` unless there is a clear reason to split it. Do not introduce a bundler or framework without asking.
- **No dependencies.** Use vanilla JS, CSS, and the Canvas and Web Audio APIs. Do not add CDN scripts or npm packages without asking.
- **Style.** Match the existing code: 2-space indentation, single quotes in JS, `const` / `let` (no `var`), arrow functions for callbacks, and `'use strict'` inside the IIFE.
- **Constants.** Tunable values go in the existing config objects (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `BOSS_PATTERNS`) rather than being hard-coded in logic.
- **Persistence.** New saved settings use the `skyfighter.` key prefix and must be wrapped so that a blocked or unavailable `localStorage` does not break the game.
- **Audio.** Audio is created only after a user gesture. Keep new sounds on the existing `beep` / `noiseBurst` helpers.
- **Accessibility.** Respect the `reducedMotion` setting for new screen shake, flashes, or particle effects.

## Making changes

1. Read the relevant section of `index.html` before editing. The file is large, so use targeted reads or search for identifiers from the table above.
2. Make the smallest change that does the job. Do not reformat unrelated code.
3. Verify by loading the game in a browser and playing through the affected path (start, fire, bomb, pause, game over, and boss fights if combat logic changed).
4. Check at phone width as well as desktop, since the layout and touch controls are responsive.

## Git

- Development branch: `claude/gifted-tesla-tv4jv1`. Do not push to other branches without explicit permission.
- Commit messages are short imperative summaries, for example `Add Sky Fighter: Raiden-style vertical shoot-em-up game`.
- Do not open pull requests unless explicitly asked.
