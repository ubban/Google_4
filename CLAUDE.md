# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo contains **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs as a
single-page HTML5 canvas app. There is no build step, no package manager, and no test suite.

- `index.html` — the entire game: markup, CSS (inline `<style>`), and JavaScript (one inline
  `<script>` wrapped in an IIFE with `'use strict'`).
- `README.md` — minimal placeholder; the title is `Google_4`.

To run the game, open `index.html` in a browser. No server is required.

## Repository layout and conventions

- Keep the game as a single self-contained HTML file unless the user asks for a split. Do not
  introduce a bundler, framework, or npm dependencies without an explicit request.
- Do not add external CDN scripts or stylesheets. The page currently loads nothing external;
  the favicon is an inline SVG data URI.
- Keep the code in the existing IIFE. Don't leak new globals onto `window`.
- Game tuning lives in top-level `const` objects inside the script. Prefer editing those over
  hard-coding numbers in logic:
  - `DIFFICULTIES` — spawn rates, enemy speed and fire chance, lives, bombs, boss HP multiplier.
  - `KILLS_PER_STAGE` — kills needed to advance a stage.
  - `THEMES` — sky gradient, cloud and sun colors per stage (cycled by stage).
  - `ENEMY_TYPES` — size, HP, speed, colors, score, and the minimum stage each appears in.
  - `BOSSES` — name, colors, and HP multiplier; `BOSS_PATTERNS` holds the bullet patterns.
- Persistence uses `localStorage` with keys prefixed `skyfighter.`:
  `bestScore`, `muted`, `difficulty`, `reducedMotion`. Wrap new storage access in try/catch
  if it can run in private or restricted browsing contexts, and keep new keys under that prefix.
- Logical canvas size is `BASE_W` x `BASE_H` (480 x 720). `resize()` scales it to the viewport;
  keep coordinates in logical units.

## Controls (keep docs and code in sync)

| Action        | Keyboard                                 | Touch / mouse                       |
|---------------|-------------------------------------------|-------------------------------------|
| Move          | Arrow keys or WASD                        | Virtual joystick (left zone)        |
| Fire          | Hold Space                                | Fire button (hold)                  |
| Smart bomb    | X or Shift                                | Bomb button                         |
| Pause         | P or Escape                               | Pause button (`#pauseBtn`)          |
| Mute / motion | On-screen icon buttons (`#muteBtn`, `#motionBtn`) | Same                        |

If you change a binding, update the handlers (`keydown` / `keyup` near the top of the input
section), the on-screen hints, and this table.

## Game loop and state

- `loop(now)` runs via `requestAnimationFrame`, clamps `dt` to 0.05 s, and calls `update(dt)`
  then `draw()` only while `running && !paused`.
- `trackPerf(dt)` adapts a `quality` level (0–2) from the average frame time over 45 frames.
  Rendering code should respect `quality` when adding expensive effects.
- `resetGame()` initializes per-run state. Add new per-run variables there so restarts are clean.
- `reducedMotion` (user setting, defaults to the OS `prefers-reduced-motion`) should gate
  screen shake and other non-essential motion.

## Editing guidelines

- Preserve the visual style: pill-style HUD, glassmorphic overlay card, and the existing color
  palette. Match the surrounding code style (2-space indent, single quotes in JS, arrow
  functions for callbacks).
- Keep the layout working at phone width with `touch-action: none` on the canvas and controls.
- Accessibility: announcements go through the `srLive` live region via `announce()`. Use it for
  new state changes that a screen-reader user would need to hear.
- Don't remove the `user-select: none` / `touch-action` settings; they prevent accidental
  selection and scrolling during play.

## Verifying changes

There is no automated test or lint setup. To verify a change:

1. Open `index.html` in a desktop browser and play through at least one stage transition and
   one boss fight after touching gameplay code.
2. Check the browser console for errors on load and during play.
3. Test touch controls with device emulation if the change touches input or layout.
4. Confirm pause, mute, difficulty, and best-score persistence still work after a reload.

If you add a test or build step, document the command here.

## Git workflow

- Default working branch for automated sessions: `claude/gifted-tesla-v5yc9h`.
- Commit messages should be short and imperative, with a body for anything non-trivial.
- Don't open pull requests unless the user asks for one.
