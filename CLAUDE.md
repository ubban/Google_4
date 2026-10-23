# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a single-page browser game called **Sky Fighter**, a Raiden-style vertical shoot-'em-up. The whole game lives in one file:

- `index.html` (~1,670 lines): inline `<style>` (lines ~10–359), HTML markup (`<body>` starting ~line 361), and one inline `<script>` (~line 422 to end) wrapped in an IIFE with `'use strict'`.
- `README.md`: minimal placeholder.

There is no build step, no bundler, no package manifest, no test suite, and no linter configured. Nothing is installed or compiled; the file runs directly in a browser.

## Running and verifying

- Open `index.html` directly in a browser, or serve the directory with any static server (for example `python3 -m http.server` from the repo root) and visit it.
- There are no automated tests. Verify gameplay changes by playing in a browser: start a game, check the affected mechanic, and confirm the HUD, overlay, and touch controls still behave.
- After any JS edit, check that the script still parses (for example, extract the `<script>` block and run `node --check` on it) since nothing else will catch syntax errors.

## Code structure (inside the `<script>` IIFE)

The script is organized into commented sections. Keep new code in the matching section:

- **DOM references**: `const xxxEl = document.getElementById(...)` at the top. Add new element lookups here, not inline.
- **Storage keys**: `STORAGE_*` constants use the `skyfighter.*` prefix for `localStorage`.
- **Adaptive performance**: `trackPerf` adjusts a `quality` level (0–2) from frame times. Visual effects should respect `quality` where they are expensive.
- **Config tables**: `DIFFICULTIES`, `THEMES` (stage backgrounds, cycled per stage), `ENEMY_TYPES`, `BOSSES`, and `BOSS_PATTERNS`. Balance changes usually belong here rather than in logic.
- **Audio**: Web Audio helpers (`ensureAudio`, `beep`, `noiseBurst`, `EXPLODE_SFX`). Audio is created lazily on the first user gesture via `ensureAudio()`; keep it that way to satisfy browser autoplay rules.
- **Input**: keyboard (arrows/WASD, Space fire, X/Shift bomb, P/Esc pause), mouse, and touch joystick plus on-screen FIRE/BOMB buttons.
- **Game state and loop**: `resetGame`, `update(dt)`, `draw()`, and `loop(now)`. `dt` is clamped to 0.05s. Game logic runs in `update`, rendering in `draw`; don't mix them.
- **Boss logic**: `spawnBoss`, `updateBoss`, `killBoss`, and the attack-pattern functions (`fireRadial`, `fireAimedSpread`, `fireSpiralBurst`, etc.).

## Conventions

- **Single file, vanilla JS, no dependencies.** Don't add libraries, a build step, or a separate module unless the user asks for it.
- **Canvas coordinates** use a fixed logical size, `BASE_W = 480` by `BASE_H = 720`, scaled by `resize()`. Gameplay positions use these logical units, not CSS pixels.
- **Accessibility is part of the feature set.** Screen-reader announcements go through `announce()` into the `#srLive` live region. Reduced-motion is user-toggleable (`reducedMotion`) and initialized from `prefers-reduced-motion`; effects such as screen shake and the flash should honor it.
- **Persist player settings through the existing `STORAGE_*` keys** and wrap any new storage reads in the same pattern used now.
- **Match the surrounding style**: two-space indentation, single quotes in JS, short section comments with `// ---------- Name ----------` headers, and `const` by default.
- Keep the file self-contained. Any new visual or UI element needs both its CSS in the `<style>` block and its markup in `<body>`.

## Git

- Development branch for this work: `claude/gifted-tesla-kgecrm`. Default branch: `main`.
- Commit messages are short, imperative, and describe the gameplay or code change (see existing history, e.g. "Add Sky Fighter: Raiden-style vertical shoot-em-up game").
- Don't create pull requests unless the user explicitly asks for one.
