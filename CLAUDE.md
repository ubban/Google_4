# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Sky Fighter is a Raiden-style vertical shoot-'em-up that runs in the browser. The entire game is one self-contained file, `index.html`, with inline `<style>` and an inline `<script>` that draws everything on a 2D canvas. There is no build step, no bundler, no package manager, and no test suite.

Other files:
- `README.md`: a placeholder. It contains only a title and a stray timestamp.
- `.git/`: the repository metadata.

## Running the game

Open `index.html` directly in a modern browser, or serve the directory with any static server, for example `python3 -m http.server` from the repo root, then visit `http://localhost:8000/`. There is nothing to install.

## Verifying changes

- There are no automated tests and no linter configured. Verify changes by loading the page and playing it, or with a headless browser such as Playwright.
- Because the script is a single IIFE, check syntax with a quick parse before committing, for example `node --check` on the extracted script body, or by loading the page and watching the console for errors.
- Test on both desktop (keyboard) and touch layouts when changing input or HUD code. The canvas is fixed at 480×720 and scaled to fit the viewport.

## Code structure (`index.html`)

The `<script>` block is wrapped in `(function () { 'use strict'; ... })()`. Keep new code inside that wrapper; do not add globals.

Major sections, roughly in file order:
- **DOM references**: `getElementById` lookups at the top of the script. Add new element lookups here.
- **Storage keys**: `STORAGE_*` constants for `localStorage` (best score, mute, difficulty, reduced motion). Use these constants, not string literals.
- **Audio**: `ensureAudio`, `beep`, `noiseBurst`, and the `sfx` object. Audio is synthesized with WebAudio, so there are no sound files.
- **Tuning tables**: `DIFFICULTIES` (spawn rates, enemy speed, lives, bombs, boss HP multiplier), `KILLS_PER_STAGE`, `THEMES` (background palettes per stage), `ENEMY_TYPES`, and `BOSSES`. Balance changes usually belong here.
- **Input**: keyboard handlers on `window` (arrows/WASD to move, Space or hold to fire, X/Shift for bomb, P/Escape to pause), mouse on the canvas, and touch joystick and buttons.
- **State and game loop**: `resetGame`, `update(dt)`, the draw functions (`drawPlane`, `drawBossShip`, and others), and the `loop` driven by `requestAnimationFrame`. Time is handled as a per-frame `dt`.
- **Weapons and power-ups**: `fireWeapon`, `applyPowerUp`, and `maybeSpawnPowerUp`.
- **Bosses**: `spawnBoss`, `BOSS_PATTERNS` (an array of attack functions per boss), and helpers such as `fireRadial` and `fireAimedSpread`.
- **Persistence**: best score is written to `localStorage` when a run ends. Storage calls can throw in private browsing, so wrap new storage access in try/catch.

## Conventions

- Keep everything in `index.html` unless there is a clear reason to split it. If you do split it, update this file.
- Use plain JavaScript (ES2017+ is fine, as the existing code uses arrow functions, `const`/`let`, and template literals). Do not add dependencies or CDN scripts without a reason.
- Match the existing style: two-space indentation, single quotes in JS, and section comments in the form `// ---------- Name ----------`.
- Respect the `reducedMotion` setting. Any new screen shake, flash, or animation should check it, as `screenShake` and `flashHit` already do.
- Accessibility matters here: update the `srLive` live region through `announce()` for significant game events, and keep HUD text readable against the sky backgrounds.
- Keep the game playable on phones: maintain the `touch-action: none` behavior and the existing touch controls.

## Git

- Development branch: `claude/gifted-tesla-vp9q0a`. Commit and push there.
- Use short, imperative commit messages that describe the gameplay or code change.
- Do not open pull requests unless asked.
