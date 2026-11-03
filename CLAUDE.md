# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo is **Sky Fighter**, a Raiden-style vertical shoot-em-up played in the browser. The entire game is a single self-contained file, `index.html`:

- Inline `<style>` for the HUD, overlays, and touch controls.
- Markup for the HUD, overlay card, pause/mute/motion buttons, difficulty buttons, and touch controls.
- One inline `<script>` wrapped in an IIFE (`'use strict'`) that drives a `<canvas id="gameCanvas">` at a fixed logical size of **480×720** (`BASE_W`, `BASE_H`), scaled to fit the window.

There is no build step, package manager, bundler, or test suite. `README.md` is a one-line title.

## Running and checking changes

- Open `index.html` directly in a browser, or serve the directory with any static server (e.g. `python3 -m http.server`).
- There are no automated tests or linters configured. Verify changes by playing the game in a browser, including the affected difficulty, weapon, boss, or touch path.
- For a headless check, a Playwright/Chromium setup can load the file and read the canvas or DOM. No such script is committed.

## Code layout inside `index.html`

Roughly in source order inside the IIFE:

1. **DOM references** — `const xxxEl = document.getElementById(...)`. Keep new HUD elements here.
2. **Storage keys** — `STORAGE_BEST`, `STORAGE_MUTE`, `STORAGE_DIFF`, `STORAGE_REDUCED` (`skyfighter.*` in `localStorage`).
3. **Adaptive performance** — `trackPerf` sets a `quality` level (0–2) from frame times.
4. **Config tables** — `DIFFICULTIES` (spawn rates, enemy speed/fire chance, lives, bombs, boss HP multiplier), `KILLS_PER_STAGE`, `THEMES` (per-stage sky palettes), `ENEMY_TYPES` (grunt, interceptor, tank), `BOSSES` (name, colors, `hpMul`).
5. **Audio** — `ensureAudio`, `beep`, `noiseBurst`, and an `sfx` object. Audio is created lazily on first user gesture (`ensureAudio()`).
6. **UI helpers** — hearts/bombs rendering, flashes, combo pops, stage banner, screen shake, mute/motion/difficulty toggles.
7. **Input** — keyboard (`keys[e.code]`; arrows/WASD move, Space fires, P/Escape pause), touch joystick and fire/bomb buttons, and mouse on the canvas.
8. **Game state and loop** — `resetGame`, `update(dt)`, `draw()` and its `draw*` helpers, `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`, `applyPowerUp`, `useBomb`, `fireWeapon`.
9. **Boss patterns** — `BOSS_PATTERNS` is an array indexed by boss, each entry an array of pattern functions built from `fireRadial`, `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`, `fireCross`, `fireHomingSeeded`, `fireDoubleRing`.
10. **Main loop** — `loop(now)` clamps `dt` to 0.05 s, calls `update` and `draw` only while `running && !paused`, then `requestAnimationFrame(loop)`.

When adding content, follow the existing table-driven pattern: add an entry to `ENEMY_TYPES`, `BOSSES`, `THEMES`, or `DIFFICULTIES` rather than hard-coding special cases in `update`/`draw`.

## Conventions

- Keep everything in `index.html` unless there is a clear reason to split files. Do not introduce a build system or framework without asking.
- Use `const` by default and `let` only for mutable game state, matching the existing style.
- Persisted values go through the `STORAGE_*` constants. Wrap any new storage access so the game still works if `localStorage` is unavailable.
- Preserve accessibility hooks: announce state changes through `announce()` (the `srLive` region), and respect the `reducedMotion` flag for shakes and flashes.
- Keep touch and keyboard controls working together; any new action needs both a key binding and a touch/button path where appropriate.
- Keep the fixed logical resolution (480×720) in mind for any new positions, sizes, or bullet speeds.

## Git

- Develop on the branch assigned to the session (currently `claude/gifted-tesla-ejymf5`) and push with `git push -u origin <branch>`.
- Write descriptive commit messages that explain the gameplay or code change.
- Do not open a pull request unless explicitly asked.
