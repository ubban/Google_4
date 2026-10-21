# CLAUDE.md

Guidance for Claude Code when working in this repository.

## What this repo is

A single-file browser game, **Sky Fighter** (a Raiden-style vertical shoot-'em-up). Everything lives in `index.html`:

- `<style>` (about lines 10–361): layout, HUD, overlay, touch controls, reduced-motion and mobile rules.
- `<body>` markup (about lines 362–421): `#container`, `<canvas id="gameCanvas">`, HUD, boss bar, touch controls, start/pause overlay.
- One inline `<script>` (about lines 422–1667) wrapped in an IIFE with `'use strict'`. All game state and logic is private to that closure.

`README.md` is a stub. There is no build step, package manager, test suite, or linter. No dependencies are loaded from a CDN.

## Running it

Open `index.html` in a browser. A static server also works, for example `python3 -m http.server` from the repo root. Do not add a build step unless asked.

## Code layout inside the script

Sections are marked with `// ---------- Name ----------` comments. Keep new code in the matching section.

| Area | Key identifiers |
| --- | --- |
| DOM handles | `canvas`, `ctx`, `overlay`, `scoreEl`, `startBtn`, ... (grabbed once at the top) |
| Persistence | `STORAGE_BEST`, `STORAGE_MUTE`, `STORAGE_DIFF`, `STORAGE_REDUCED` (`skyfighter.*` keys in `localStorage`) |
| Adaptive quality | `quality` (0 full, 1 medium, 2 low), `trackPerf(dt)` lowers quality when frames average under 40 fps and raises it above 56 fps |
| Tuning data | `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES` (per-stage sky palettes), `ENEMY_TYPES`, `BOSSES` |
| Audio | `ensureAudio()`, `beep()`, `noiseBurst()`, `EXPLODE_SFX` (Web Audio, created lazily on first user gesture) |
| Input | `keys` map (keyboard), joystick helpers (`joystickStart/Move/End`), `canvasPointFromEvent` (mouse and touch) |
| Game logic | `resetGame`, `update(dt)`, `spawnEnemy`, `spawnBoss`, `applyPowerUp`, `useBomb`, `fireWeapon`, `killBoss`, `loseLife`, `endGame`, `togglePause` |
| Boss attacks | `BOSS_PATTERNS` and the `fire*` helpers (`fireRadial`, `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`, `fireCross`, `fireHomingSeeded`, `fireDoubleRing`) driven by `updateBoss(dt)` |
| Rendering | `resize()` (scales a fixed `BASE_W × BASE_H = 480 × 720` logical canvas), `drawPlane`, `drawBossShip`, `drawClouds`, `draw()` |
| Main loop | `loop(now)` calls `trackPerf`, then `update(dt)` and `draw()` while running and not paused. `dt` is clamped to 50 ms. |

## Conventions

- **Plain JavaScript and DOM APIs only.** No frameworks, no bundler, no modules. Keep everything inside the existing IIFE and `<script>` tag.
- **Match the existing style.** Two-space indentation, single quotes, `const` by default, arrow functions for callbacks, and short `// ----- Section -----` banners. Comments explain why, not what.
- **Frame-rate independence.** Movement and timers use `dt` in seconds. Do not hard-code per-frame increments.
- **Keep the logical resolution.** Gameplay coordinates are in the 480 × 720 space. Do not depend on CSS pixel size.
- **Respect reduced motion.** Screen shake, flash, and pulse effects must check `reducedMotion` (body class `reduced-motion`). New visual effects should follow the same rule.
- **Respect mute.** Route sound through the existing helpers so `muted` is honored.
- **Accessibility.** Announce important state changes through `announce()` (the `#srLive` region). Buttons need `title` or visible text.
- **Persist only through the `skyfighter.*` keys.** Wrap new `localStorage` use in the same simple pattern as the existing code.
- **Touch.** Canvas and control zones use `touch-action: none`. Keep it that way so gestures don't scroll the page.

## Testing and verification

There is no automated test suite. Verify changes by:

1. Loading `index.html` in a browser and checking the console for errors (`node --check` can be used on the extracted script for a quick syntax check).
2. Playing through the affected path: start, fire, bomb, pause/resume, lose a life, game over, and a stage transition (`KILLS_PER_STAGE` kills). Boss changes need a boss fight.
3. Checking all three difficulties, keyboard, mouse, and touch controls when input code changes.
4. Checking reduced-motion and mute toggles when effects or audio change.

## Git

- Develop on the branch named in the task (currently `claude/gifted-tesla-buftk3`). Do not push to other branches without permission.
- Write clear commit messages that describe the change.
- Do not open a pull request unless explicitly asked.
