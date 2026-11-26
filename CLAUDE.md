# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a small static web project. Its only real content is **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs in the browser on an HTML5 canvas.

| File        | Purpose                                                        |
|-------------|----------------------------------------------------------------|
| `index.html`| The entire game: markup, CSS (`<style>`), and JS (`<script>`) in one file (~1,670 lines). |
| `README.md` | Minimal project readme.                                        |

There is no build step, package manager, bundler, test suite, linter, or CI configuration. Do not invent one. Open `index.html` directly in a browser, or serve the directory with any static server, to run the game.

## Commands

None are defined. If you add tooling later, document the commands here.

## Architecture of `index.html`

- **Structure**: `<style>` in the head (lines ~10–362), HTML body markup including the `<canvas id="gameCanvas">` (~363), then one `<script>` (~422 onward) wrapped in an IIFE with `'use strict'`. Nothing is exported to the global scope.
- **Rendering**: 2D canvas. The logical play field is `BASE_W = 480` × `BASE_H = 720`, scaled to the viewport by `resize()`.
- **Loop**: `loop(now)` runs through `requestAnimationFrame`, clamps `dt` to 50 ms, then calls `update(dt)` and `draw()` while the game is running and not paused.
- **Adaptive quality**: `trackPerf()` keeps a 45-frame rolling average and steps `quality` between 0 (full), 1 (medium), and 2 (low) to protect frame rate.
- **State**: module-level `let` variables inside the IIFE. A reset goes through `resetGame()`.
- **Difficulty**: the `DIFFICULTIES` table (`easy`, `normal`, `hard`) sets spawn rates, enemy speed, enemy fire chance, lives, bombs, and boss HP multiplier.
- **Content tables**: `THEMES` (background palettes cycled per stage), `ENEMY_TYPES` (`grunt`, `interceptor`, `tank`), `BOSSES` (six bosses, cycled), and `BOSS_PATTERNS` (bullet patterns built from `fire*` helpers).
- **Progression**: `KILLS_PER_STAGE = 18` advances the stage. Enemies gate on `minStage`.
- **Audio**: Web Audio synthesis only (`beep`, `noiseBurst`, `ensureAudio`). No audio files. Audio is created lazily on the first user gesture.
- **Persistence**: `localStorage` keys are prefixed `skyfighter.` (`bestScore`, `muted`, `difficulty`, `reducedMotion`). Every read should tolerate missing or blocked storage, and the game should still run without it.
- **Input**: keyboard (`keys` map on `keydown`/`keyup`), mouse on the canvas, and a touch joystick plus fire and bomb buttons for mobile.
- **Accessibility**: `announce()` writes to a `aria-live` region (`srLive`). Respect `prefers-reduced-motion` through the `reducedMotion` flag and the `reduced-motion` body class.

## Conventions

- Keep the game self-contained in `index.html`. Do not add external CDN scripts or stylesheets unless the user asks for them.
- Keep the IIFE and `'use strict'` wrapper. Don't leak globals.
- Use the existing `STORAGE_*` constants for `localStorage` keys, and keep the `skyfighter.` prefix.
- New enemies, bosses, or themes go in the existing tables (`ENEMY_TYPES`, `BOSSES`, `THEMES`) rather than as hard-coded branches in `update()`/`draw()`.
- Use `dt`-based movement (seconds) for anything that should be frame-rate independent.
- Keep the layout working on phone widths. Touch controls are shown on touch devices, and the canvas must stay usable under the viewport resize logic.
- Preserve the existing 2-space indentation and the section comments (`// ---------- Name ----------`).

## Verification

Since there is no test harness, check changes by:

1. Opening `index.html` in a browser (or a static server) and confirming there are no console errors.
2. Playing through start, a stage transition, a boss fight, bomb use, pause/resume, and game over.
3. Checking the difficulty buttons, the mute and reduced-motion toggles, and that the best score persists across reloads.
4. Checking touch controls at a phone-sized viewport if the change touches input or layout.

For visual changes, take a screenshot before and after.

## Git workflow

- Development branch: `claude/gifted-tesla-y3k6p4`. Commit and push there. Do not push to other branches without explicit permission.
- Do not open a pull request unless the user explicitly asks for one.
- Keep commits focused. Write messages that describe the gameplay or code change, not the tooling used.
