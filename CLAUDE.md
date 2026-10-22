# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo is a single-page HTML5 canvas game, **Sky Fighter** — a Raiden-style vertical shoot-em-up.

- `index.html` — the entire game: markup, CSS, and one inline `<script>` (an IIFE). ~1,670 lines.
- `README.md` — minimal placeholder.

There is no build step, no package manager, no bundler, and no test suite. Open `index.html` in a browser to run it.

## Working on the game

Source layout inside `index.html`, top to bottom:

1. **Styles** (`<style>`, ~lines 10–360): HUD pills, glassmorphic overlay card, touch controls, reduced-motion and vignette styles.
2. **Markup** (~lines 360–420): HUD elements, overlay, touch controls, and a single `<canvas id="gameCanvas">`.
3. **Script** (`<script>`, ~line 422 onward), wrapped in `(function () { 'use strict'; ... })()`. Sections, in order:
   - DOM lookups and `STORAGE_*` keys
   - Adaptive performance (`quality` 0–2, driven by frame times)
   - Config constants: `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES`, `ENEMY_TYPES`, `BOSSES`
   - Audio (`ensureAudio`, `beep`, `noiseBurst`, `sfx`)
   - HUD helpers and effects (hearts, bombs, combo pops, stage banner, screen shake)
   - Input (keyboard, touch joystick, mouse)
   - Game state (`resetGame`), spawning, bullets, bombs, and power-ups
   - Boss patterns (`BOSS_PATTERNS`, `updateBoss`, `killBoss`)
   - `update(dt)`, `draw()`, and the `loop(now)` requestAnimationFrame driver

Keep changes inside the existing IIFE. Do not introduce globals.

### Conventions to keep

- **Tuning lives in constants.** Difficulty, enemy stats, boss HP multipliers, and stage pacing belong in the `DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, and `KILLS_PER_STAGE` tables rather than hard-coded in logic.
- **Persistence uses `localStorage` with `skyfighter.*` keys.** Current keys: `bestScore`, `muted`, `difficulty`, `reducedMotion`. Reads have fallbacks, so keep that pattern when adding new keys.
- **Accessibility hooks matter.** Use `announce()` for screen-reader updates, and respect `reducedMotion` for shake/flash effects.
- **Mobile is first-class.** Touch controls are shown when `isTouchDevice` is true. Any new control needs a touch path as well as keyboard/mouse.
- **Scale to the base resolution.** Game logic uses `BASE_W = 480`, `BASE_H = 720`; `resize()` handles the canvas fit.

### Controls (for manual testing)

| Action | Keyboard | Touch |
| --- | --- | --- |
| Move | Arrows / WASD | Left joystick zone |
| Fire | Space (hold) | Fire button (hold) |
| Bomb | X or Shift | Bomb button |
| Pause | P or Escape | Pause button |
| Mute / reduced motion | — | HUD buttons |

## Testing

There are no automated tests or linters. Verify changes by:

1. Opening `index.html` in a browser (or serving the directory with any static server).
2. Playing through at least one stage transition and one boss fight, since those touch the most code paths.
3. Checking all three difficulties, the mute and pause toggles, and the best-score persistence across reloads.
4. Checking the touch layout at phone width (or with device emulation) when touching HUD or controls.

For changes to rendering or audio, also watch the browser console for errors.

## Git

- Development branch for this work: `claude/gifted-tesla-vo324s`.
- Commit messages in this repo are imperative and descriptive, with a summary line followed by a bulleted body for larger changes.
- Do not open pull requests unless explicitly asked.
