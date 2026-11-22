# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a single-page browser game, **Sky Fighter**: a Raiden-style vertical shoot-'em-up rendered on an HTML `<canvas>`.

- The entire game (HTML, CSS, and JavaScript) lives in **`index.html`** (~1,670 lines). There is no bundler, framework, package manager, or build step.
- `README.md` is a placeholder and contains no setup instructions.
- There is no test suite, linter, or CI configuration in the repo.

## Running it

No build is needed. Open `index.html` directly in a browser, or serve the directory with any static server, for example:

```bash
python3 -m http.server 8000   # then visit http://localhost:8000/
```

## Code layout (`index.html`)

- **`<head>`**: meta tags (viewport locked to device width, dark `theme-color`), an inline SVG favicon, and an inline `<style>` block. Colors and layout for the HUD, overlay, boss bar, stage banner, vignette, and touch controls are all defined here.
- **`<body>`**: the HUD (score, best, multiplier, weapon, hearts, bombs), the boss bar, the stage banner, the start/pause overlay, the touch controls (joystick, fire, bomb), and the canvas `#gameCanvas`.
- **`<script>`** (starts near line 425): one IIFE-style block with these sections, roughly in order:
  - DOM handles and `STORAGE_*` keys (`localStorage` keys are prefixed `skyfighter.`).
  - Perf tracking (`trackPerf`) that drops render quality (`quality` 0/1/2) when frames are slow.
  - Tuning tables: `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES` (per-stage sky palettes), `ENEMY_TYPES`, and `BOSSES`.
  - Audio: a lazily created `AudioContext` (`ensureAudio`) and synthesized sound helpers (`beep`, `noiseBurst`, `sfx`). No audio files are loaded.
  - HUD helpers (`renderHearts`, `renderBombs`, `showStageBanner`, `showComboPop`, `screenShake`).
  - Responsive scaling (`resize`) against a fixed logical canvas of `BASE_W = 480` × `BASE_H = 720`.
  - Input: keyboard (`keys`, keydown/keyup), touch joystick and buttons, and mouse aiming on the canvas.
  - Game state and loop: `update(dt)`, `draw()`, `useBomb()`, `togglePause()`, and `loop(now)` driven by `requestAnimationFrame`.

## Gameplay and controls

- **Move**: arrow keys / WASD, mouse-follow on the canvas, or the on-screen joystick on touch devices.
- **Fire**: held fire input (Space, mouse button, or the fire button on touch).
- **Smart bomb**: `X` or Shift, or the bomb button.
- **Pause**: `P` or Escape.
- **Difficulty** (`easy` / `normal` / `hard`) changes spawn rate, enemy shooting, enemy speed, starting lives and bombs, and boss HP multiplier.
- **Progression**: each stage needs `KILLS_PER_STAGE` (18) kills before a boss fight. Stages cycle through `BOSSES` and `THEMES`. Enemy types unlock by stage (`minStage`).

## Conventions and guidelines

- **Keep it a single file.** Unless the user asks otherwise, add new markup, styles, and scripts to `index.html` rather than introducing separate asset files or a build system.
- **Stay dependency-free.** Do not add external scripts, CDNs, or frameworks. Audio is synthesized with the Web Audio API, and graphics are drawn with the 2D canvas API.
- **Preserve the tuning tables.** Balance lives in `DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, and `THEMES`. Change values there rather than hard-coding special cases in the loop.
- **Respect the reduced-motion and mute settings.** Visual effects (screen shake, stage pulse, heart beat) should honor `reducedMotion` and `muted`, which persist via `localStorage`.
- **Mobile and touch matter.** The page is locked to `user-scalable=no` and `touch-action: none`. Any new control must work with touch and respect `env(safe-area-inset-*)` padding, as the HUD already does.
- **Keep the HUD accessible.** Announcements go through the `announce()` helper and the `#srLive` live region.
- **Use the logical canvas size.** Positions and sizes are in `BASE_W` × `BASE_H` units; scaling is handled by `resize()`, so avoid using raw window pixels in game logic.

## Verifying changes

Since there are no automated tests, verify changes manually:

1. Open `index.html` in a browser (or serve it locally) and confirm there are no console errors.
2. Start a game, then check movement, firing, bombs, pause, and game over.
3. Reach a stage boss, or temporarily adjust `KILLS_PER_STAGE` locally, to exercise boss and stage transitions.
4. Check the layout at phone width and in touch mode if the change touches the HUD or controls.
5. Toggle mute and reduced motion and confirm the settings persist after reload.

## Git workflow

- Develop on the branch `claude/gifted-tesla-3gazge`. Do not push to other branches without explicit permission.
- Do not open a pull request unless the user asks for one.
