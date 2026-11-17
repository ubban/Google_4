# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a small static web project. Its only real content is **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs as a single self-contained HTML5 canvas page.

## Repository layout

| Path | Purpose |
| --- | --- |
| `index.html` | The entire game: HTML markup, inline `<style>`, and one inline `<script>` (~1,670 lines). |
| `README.md` | Placeholder title only. |
| `CLAUDE.md` | This file. |

There is no build system, package manager, test suite, linter, or bundler. Do not introduce one unless asked.

## Running the game

Open `index.html` directly in a browser, or serve the directory with any static server (for example `python3 -m http.server`). No install step is needed.

## Architecture of `index.html`

- **Single IIFE.** All game code lives inside `(function () { 'use strict'; ... })()` in the one inline script. Nothing is exported to `window`.
- **Rendering.** One `<canvas id="gameCanvas">` drawn with the 2D context. The logical play field is `BASE_W = 480` by `BASE_H = 720`; `resize()` scales it to fit the viewport. Game state is updated in `update(dt)` and drawn each frame from a `requestAnimationFrame` loop.
- **HUD and overlays.** Score, best score, multiplier, weapon, lives (hearts), bombs, boss bar, combo pop, and stage banner are DOM elements layered over the canvas, not drawn on it. The start/game-over card is `#overlay`.
- **Audio.** Synthesized with the Web Audio API (`beep`, `noiseBurst`, `ensureAudio`). No audio files.
- **Accessibility.** A `#srLive` live region is used by `announce()`. A reduced-motion mode (`body.reduced-motion`) softens screen shake and turns off several animations (vignette, heart beat, banners, overlay entrance); it defaults to the OS `prefers-reduced-motion` setting.
- **Touch support.** A virtual joystick (`#moveZone`), a FIRE button, and a BOMB button are shown for touch devices.

### Key tunables (all near the top of the script)

- `DIFFICULTIES`: `easy` / `normal` / `hard`. Each sets spawn rate, enemy speed, enemy fire chance, starting lives, starting bombs, and boss HP multiplier.
- `KILLS_PER_STAGE = 18`: kills needed before a stage boss spawns.
- `THEMES`: six sky palettes, cycled per stage.
- `ENEMY_TYPES`: `grunt`, `interceptor`, `tank`, with `minStage` gating when each appears.
- `BOSSES`: six bosses, each with a name, palette, and HP multiplier. Attack patterns are in `BOSS_PATTERNS` (radial, aimed spread, columns, spiral, cross, homing, double ring).

### Persistence

`localStorage` keys, all prefixed `skyfighter.`:

- `skyfighter.bestScore`
- `skyfighter.muted`
- `skyfighter.difficulty`
- `skyfighter.reducedMotion`

Any change to these keys or their formats must keep existing saved values readable.

### Controls

- Keyboard: arrows or WASD to move, Space to fire, X or Shift for a bomb, P or Esc to pause.
- Mouse: move to steer, hold click to fire.
- Touch: drag on the left side to fly, tap FIRE, tap BOMB.

Keep the on-screen `#controlsHint` text in sync when controls change.

## Conventions

- **Keep it a single file** unless the user asks for a split. Styles and script stay inline in `index.html`.
- **No external dependencies.** Do not add CDN scripts, fonts, or npm packages without asking. The page must work offline.
- **Match the existing style:** 2-space indentation, `const` by default, arrow functions for callbacks, and short helper functions with descriptive names.
- **Preserve the game's feel.** Balance numbers live in the tunable constants above. Change them deliberately and say what changed.
- **Don't break mobile.** The layout uses `viewport-fit=cover`, safe-area insets, and `touch-action: none`. Verify changes to HUD or controls at phone width.

## Verifying changes

There are no automated tests. For any gameplay or UI change:

1. Open `index.html` in a browser, or use the `run` skill to launch it.
2. Check the affected flow end to end: start, play, pause, bomb, stage transition, boss, and game over.
3. Check at a phone-sized viewport as well as desktop.
4. Confirm that `localStorage` values still load after a reload.

Syntax check the inline script if needed, for example by extracting the `<script>` block and running `node --check` on it.

## Git

- Development branch: `claude/gifted-tesla-wwib4t`. Do not push to other branches without explicit permission.
- Commit messages are short imperative summaries. The existing history uses a one-line subject followed by a bulleted body for larger changes.
- Do not open a pull request unless the user asks for one.
