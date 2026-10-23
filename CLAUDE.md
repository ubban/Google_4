# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a single-page HTML5 canvas game, **Sky Fighter**: a Raiden-style vertical shoot-'em-up. The whole game lives in `index.html`:

- `<style>`: all CSS (HUD, overlay card, touch controls, reduced-motion rules, theme tokens).
- `<body>`: static markup for the HUD, overlay, pause/mute/motion buttons, touch controls, and a `<canvas id="gameCanvas">`.
- A single inline `<script>` wrapped in an IIFE with `'use strict'`. Nothing is exposed on `window`.

`README.md` is a placeholder. There is no other source code.

## Running and testing

- There is no build step, package manifest, bundler, linter, or test suite. Do not add one unless asked.
- To run the game, open `index.html` in a browser (or serve the directory with any static server). Reloading is the whole dev loop.
- To verify a change, play it in a browser: start a run, exercise the feature, and check the console for errors. Test touch controls with device emulation when a change touches input or layout.

## Code structure inside the script

The script is organized by `// ----------` section comments and `const` blocks. Keep new code in the matching section.

- **DOM refs**: all `document.getElementById` lookups happen at the top of the IIFE.
- **Storage keys**: `STORAGE_*` constants, all prefixed `skyfighter.`. Best score, mute, difficulty, and reduced-motion are persisted in `localStorage`.
- **Config tables**: `DIFFICULTIES`, `THEMES` (stage backdrops), `ENEMY_TYPES`, `BOSSES`, `BOSS_PATTERNS`. Tune balance here rather than in logic.
- **Audio**: `ensureAudio()` lazily creates the `AudioContext` on the first user gesture. Sound effects are built from `beep()`, `noiseBurst()`, and the `sfx` helpers. Mute is applied through `muted`.
- **Input**: keyboard (`keys[e.code]`), mouse on the canvas, and touch joystick plus fire/bomb buttons. Keep all three working when you change controls.
- **Game loop**: `update(dt)` advances state, `draw()` renders, and `loop(now)` drives both through `requestAnimationFrame`. `resetGame()` sets up a new run.
- **Adaptive quality**: `trackPerf()` moves a `quality` level (0–2) based on average frame time. Respect this when adding expensive effects.
- **Bosses**: `spawnBoss()`, `updateBoss()`, `killBoss()`, and the `fire*` pattern helpers. Patterns are picked from `BOSS_PATTERNS`.
- **Accessibility**: `announce()` writes to the `srLive` live region. Use it for important state changes (stage clear, game over). Reduced motion is toggled with `motionBtn` and adds the `reduced-motion` body class. Screen shake and flashes should respect it.

## Conventions

- Match the existing style: 2-space indentation, single quotes in JS, `const` by default, arrow functions for callbacks, and short section comments.
- Keep everything in `index.html` unless the user asks for a split. Do not introduce frameworks, a bundler, or external CDN dependencies.
- The file uses a fixed logical resolution of 480×720 (`BASE_W`, `BASE_H`), scaled by `resize()`. Keep gameplay coordinates in that space.
- Keep the viewport meta, the `theme-color` meta, and `:root { color-scheme: dark; }` intact.
- Keep the layout usable at phone width. The page disables user scaling and uses `viewport-fit=cover`, so respect safe-area insets.
- Do not add comments that restate what the code does. Comment non-obvious game-balance or browser-quirk reasoning.

## Git workflow

- Develop on the branch named in the session instructions. Do not push to `main` directly.
- Commit with clear, descriptive messages that explain the gameplay or UI change.
- Do not open pull requests unless explicitly asked.
