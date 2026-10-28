# CLAUDE.md

Guidance for working in this repository.

## What this is

A single-file browser game, **Sky Fighter** (a vertical shoot-em-up in the style of Raiden). Everything lives in `index.html`: markup (`<body>`, ~line 361), CSS (`<style>`, lines 10–360), and one inline `<script>` IIFE (line 422 onward). `README.md` is a placeholder.

There is no build step, package manager, bundler, linter, or test suite. Do not add one unless asked.

## Running it

Open `index.html` in a browser. No server is needed. For a quick local server, use `python3 -m http.server` from the repo root.

## Verifying changes

There are no automated tests, so check changes by hand in a browser:

- Play through at least one stage, including a stage-boss fight.
- Check all three difficulties (Easy / Normal / Hard) if you touched difficulty values.
- Check pause (`P` or `Esc`), mute, and reduced-motion toggles if you touched them.
- Check touch controls by enabling a touch device emulator, if you touched input or layout.
- Check the browser console for errors. The game runs in a single `requestAnimationFrame` loop, so an uncaught exception stops it.

## Code layout inside the script

All game code sits inside one `(function () { 'use strict'; ... })()` wrapper. Nothing is exported to `window`, so keep new helpers inside it.

Key sections, in order of appearance (search for the `// ----------` comments):

- **Element refs and storage keys**: `document.getElementById` lookups, and the `skyfighter.*` `localStorage` keys (best score, mute, difficulty, reduced motion).
- **Adaptive performance** (`trackPerf`): lowers a `quality` level (0–2) when frame time stays high. Check it before adding per-frame work.
- **Tuning tables**: `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES` (background palette per stage), `ENEMY_TYPES`, `BOSSES`. Balance changes usually belong here, not in logic.
- **Audio** (`ensureAudio`, `beep`, `noiseBurst`, `sfx`): Web Audio synthesis, so no audio files. Audio starts only after a user gesture.
- **Canvas sizing** (`resize`): fixed 480×720 logical resolution, scaled to fit the window (max 1.6×).
- **Input**: keyboard (arrows/WASD to move, Space to fire, X or Shift for bomb, P or Esc to pause), mouse/pointer, and touch joystick plus fire/bomb buttons.
- **Game state**: `resetGame`, `update(dt)`, `applyPowerUp`, `useBomb`, `fireWeapon`. Power-ups are `life`, `bomb`, and `weapon` (weapon levels 1–3, then swap between VULCAN and LASER).
- **Bosses**: `spawnBoss`, `BOSS_PATTERNS` (each entry is a bullet pattern function), `updateBoss`, `killBoss`.
- **Rendering**: `draw`, `drawPlane`, `drawBossShip`, `drawClouds`. Drawing is canvas 2D only.
- **Loop**: `loop(now)` caps `dt` at 0.05 s, so pausing or tab-switching does not cause a time jump.

## Conventions

- Keep to the existing style: `const` and `let`, arrow functions for callbacks, two-space indentation, and short section comments in the `// ---------- Name ----------` form.
- Use `dt`-scaled movement (speeds × `dt`) so motion does not depend on frame rate.
- Persist settings only through the existing `skyfighter.*` `localStorage` keys. Wrap new reads and writes in the same pattern the file already uses.
- Respect the reduced-motion setting. Screen flashes are skipped when `reducedMotion` is on, and `screenShake` is damped. New flashing or shaking effects should follow the same pattern.
- Keep the play area at the logical 480×720 size. HUD and overlay styling lives in CSS, not in canvas drawing.

## Git

- Develop on the branch assigned for the session. Do not push to `main` directly.
- Write commit messages in the imperative mood, with a short subject line that says what changed.
