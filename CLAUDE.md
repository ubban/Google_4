# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

**Sky Fighter** is a single-page, Raiden-style vertical shoot-'em-up that runs in the browser on a `<canvas>`. There is no build step, no package manager, no framework, and no test suite.

Files:

- `index.html` — the entire game: markup, inline CSS, and one inline `<script>` IIFE with all game logic.
- `README.md` — minimal placeholder.

To run it, open `index.html` in a browser (or serve the directory with any static server). Nothing needs to be installed.

## Architecture (inside `index.html`)

- **Markup**: a `#container` holds the `<canvas id="gameCanvas">`, a HUD (`#hud`: score, best, multiplier, weapon, hearts, bombs), boss bar (`#bossBarWrap`), `#topBar` icon buttons (mute, reduced-motion, pause), touch controls (`#touchControls`: joystick zone, FIRE, BOMB), and `#overlay` (title / start / game-over card with difficulty buttons).
- **Script**: wrapped in `(function () { 'use strict'; ... })()`. Everything is file-private; there are no modules or exports.
- **Configuration constants** near the top of the script:
  - `DIFFICULTIES` — `easy` / `normal` / `hard` tuning (spawn rate, enemy speed, shoot chance, lives, bombs, boss HP multiplier).
  - `KILLS_PER_STAGE` — kills needed before a boss spawns.
  - `THEMES` — per-stage sky palettes and names (cycled by stage).
  - Boss designs — names, hull colors, and `hpMul`.
- **Game loop**: `loop(now)` calls `update(dt)` then `draw()`. `update` handles input, spawning, collisions, power-ups, and boss patterns. `draw` renders the scene (clouds, planes, bosses, bullets, particles).
- **Boss attacks** are separate pattern functions (`fireRadial`, `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`, `fireCross`, `fireHomingSeeded`, `fireDoubleRing`) driven by `updateBoss`.
- **Adaptive quality**: `trackPerf` samples frame times and steps a `quality` level (0–2) down when frame rate drops and back up when it recovers.
- **Audio**: a lazily created Web Audio context (`ensureAudio`). Sound effects come from `beep` / `noiseBurst` and the `sfx` helpers. Audio must start from a user gesture.
- **Input**: keyboard (arrows/WASD move, Space fire, X or Shift bomb, P/Esc pause), mouse (move to steer, click to fire), and touch (left-side joystick, FIRE and BOMB buttons).
- **Accessibility**: `#srLive` is an `aria-live` region updated through `announce()`. Reduced motion can be set by the OS preference or the in-game toggle, and it disables or softens screen shake, flashes, and pulses.

## Persistence

The game stores four keys in `localStorage`. All use the `skyfighter.` prefix:

| Key | Meaning |
| --- | --- |
| `skyfighter.bestScore` | Best score (number) |
| `skyfighter.muted` | `'1'` when muted |
| `skyfighter.difficulty` | `easy`, `normal`, or `hard` |
| `skyfighter.reducedMotion` | `'1'` / `'0'`; absent means "follow OS preference" |

When you change any of these, keep the prefix and check that existing saved values still load correctly. Reads of `localStorage` should tolerate it being unavailable (private mode, blocked storage) if you add new ones.

## Conventions

- Keep everything in `index.html` unless the user asks for a split. Do not introduce a build tool, bundler, or npm dependency without asking.
- Stay in the existing style: `const`/`let`, arrow callbacks, two-space indentation, single quotes in JS, and small named helper functions.
- Use the existing `DIFFICULTIES` and constants for tuning rather than hard-coding new magic numbers in update logic.
- Any new HUD element, button, or announcement needs an accessible name and, where it conveys state, an `announce()` call.
- Preserve the mobile layout: `viewport-fit=cover`, safe-area insets, `touch-action: none`, and the no-scroll `html, body` setup.
- Commit messages are short imperative summaries (see `git log`).

## Verifying changes

There are no automated tests or linters. To check a change:

1. Open `index.html` in a browser and confirm there are no console errors.
2. Play through at least one stage transition and one boss fight after changing gameplay code.
3. Check the HUD and overlay at a phone-sized viewport, and test touch controls if you touched input or layout code.
4. Toggle mute and reduced motion and confirm they persist across reloads.

Browser-based checks can use the pre-installed Playwright/Chromium in the cloud environment.

## Git workflow

- Develop on the branch assigned for the task (currently `claude/gifted-tesla-i98i5p`). Do not push to other branches without permission.
- Do not open a pull request unless explicitly asked.
