# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo contains **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs in the browser as a single HTML5 canvas page. There is no build step, no package manager, and no test suite.

Files:

- `index.html`: the entire game. CSS is in a `<style>` block (~lines 10–361), the markup in `<body>` (canvas, HUD, overlays, touch controls), and all JavaScript in one `<script>` IIFE starting at ~line 422.
- `README.md`: a placeholder. Keep it short.

## Running and checking changes

- Open `index.html` directly in a browser, or serve the directory with any static server (e.g. `python3 -m http.server`).
- There are no tests, linters, or build commands configured. Verify changes by loading the page and playing: start a run, fire, use a bomb, pause, switch difficulty, and check the touch controls if you touched layout or input code.
- Do not add a build tool, framework, or dependency unless the user asks for one. The single-file, zero-dependency design is intentional.

## Code structure (inside `index.html`)

The script is wrapped in `(function () { 'use strict'; ... })()`. Keep new code inside that wrapper.

- **DOM references**: a block of `const x = document.getElementById(...)` at the top of the script. Add new element lookups there.
- **Persistence**: `localStorage` keys are prefixed `skyfighter.` (`bestScore`, `muted`, `difficulty`, `reducedMotion`). Reads and writes are not wrapped in try/catch, so keep that in mind if you add more storage.
- **Tuning data** (plain objects/arrays near the top of the script): `DIFFICULTIES` (spawn rate, enemy speed, shoot chance, lives, bombs, boss HP multiplier), `KILLS_PER_STAGE`, `THEMES` (per-stage sky/cloud/sun palettes), `ENEMY_TYPES`, `BOSSES`, and `BOSS_PATTERNS`. Prefer changing these tables over hard-coding numbers in logic.
- **Audio**: a Web Audio synth (`ensureAudio`, `beep`, `noiseBurst`, and an `sfx` object). No audio files. Audio is created lazily on the first user gesture; keep it that way for autoplay policies.
- **Game loop**: `update(dt)` advances state, `draw()` renders, and `loop(now)` drives both via `requestAnimationFrame`. Keep game-state mutation in `update` and drawing in `draw`.
- **Entities**: enemies, bullets, power-ups, and explosions are plain objects in arrays, spawned by `spawn*` helpers and removed when off-screen or dead.
- **Boss fights**: `spawnBoss`, `updateBoss`, `killBoss`, and the `fire*` pattern helpers. Boss attack patterns are selected from `BOSS_PATTERNS`.
- **Input**: keyboard (arrows/WASD move, Space fire, X or Shift bomb, P or Esc pause), mouse on the canvas (move to steer, click to fire), and touch controls (virtual joystick in `moveZone`, `fireBtn`, `bombBtn`). Input state lives in `keys`, `firing`, `joyActive`/`joyVec`, and `mouseActive`. The keyboard and pointer hints are also shown in `#controlsHint`, so update it when bindings change.
- **Scaling**: the logical play field is fixed at `BASE_W = 480` × `BASE_H = 720`. `resize()` scales the canvas CSS size to fit the viewport (capped at 1.6×). Game coordinates always use the base size.
- **Accessibility**: `announce()` writes to the `srLive` live region. A reduced-motion mode (`reducedMotion`, reflected as `body.reduced-motion`) dampens screen shake (`screenShake`), skips hit/bomb flashes, and turns off some CSS animations (vignette, combo pop, stage banner, overlay card). It defaults to the OS `prefers-reduced-motion` setting until the user toggles it.

## Conventions

- Match the existing style: 2-space indentation, single quotes in JS, `const` by default, small named helper functions, and short section-like comments only where they help.
- Keep everything in `index.html` unless there is a clear reason to split it. If you do split files, reference them by relative path and keep the page working when opened from disk.
- Preserve mobile behavior: the page uses `touch-action: none`, safe-area insets, and a viewport lock. Test any input change with touch in mind.
- Don't change `localStorage` key names without a migration plan, since they hold players' saved best scores and settings.

## Git

- Development branch for this work: `claude/gifted-tesla-cdunqa`. Commit and push there; don't push to `main` without explicit permission.
- Write clear commit messages that describe the gameplay or code change.
- Don't open a pull request unless the user asks for one.
