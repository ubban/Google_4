# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` contains **Sky Fighter**, a Raiden-style vertical shoot-'em-up built as a single self-contained HTML5 canvas page.

- `index.html` — the entire game: markup, CSS (`<style>`, lines 10–359), canvas element, HUD, overlays, and all JavaScript (`<script>`, starts ~line 422).
- `README.md` — minimal project title only.

There is no build step, no package manager, no bundler, no test suite, and no linter configured. Do not add any of these unless asked.

## Running and verifying

- Open `index.html` directly in a browser, or serve the directory with any static server (e.g. `python3 -m http.server`).
- Verification is manual: play the game in a browser and check the change. There are no automated tests to run.
- The game is responsive: the canvas is a fixed logical size of `480×720` (`BASE_W`/`BASE_H`) scaled to fit the window (max 1.6×). Keep gameplay coordinates in logical units.

## Code layout inside `index.html`

The script is one IIFE with `'use strict'`. Its sections, in order:

1. **DOM references** — `document.getElementById(...)` lookups at the top. Element IDs are load-bearing; renaming an ID in markup means updating its lookup here.
2. **Storage keys** — `STORAGE_*` constants, all prefixed `skyfighter.`.
3. **Adaptive performance** — `trackPerf` samples frame time and steps a `quality` level (0–2).
4. **Tuning tables** — `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES` (per-stage sky palettes), `ENEMY_TYPES`, `BOSSES`.
5. **Persistence and settings** — best score, mute, difficulty, reduced motion.
6. **Audio** — Web Audio helpers (`ensureAudio`, `beep`, `noiseBurst`, `sfx.*`). Audio is created lazily on the first user gesture.
7. **HUD helpers** — hearts, bombs, combo pops, stage banner, screen shake, hit flash.
8. **Canvas sizing** — `resize()`.
9. **Input** — keyboard listeners, touch joystick and buttons, mouse aim on the canvas.
10. **Game state and simulation** — `resetGame`, `update(dt)`, spawning, collisions, power-ups, bombs, `fireWeapon`, score and chain (`addScore`, `breakChain`), `loseLife`, `endGame`, `togglePause`.
11. **Bosses** — `BOSS_PATTERNS` (bullet pattern functions such as `fireRadial`, `fireCross`, `fireHomingSeeded`) and `updateBoss` / `killBoss`.
12. **Rendering** — `drawPlane`, `drawBossShip`, `drawClouds`, `draw()`.
13. **Main loop** — `loop(now)` clamps `dt` to 50 ms, then calls `update` and `draw` only when `running && !paused`. `startBtn` starts a run.

Keep new code in the section it belongs to, and follow the existing `// ---------- Section ----------` comment style.

## Conventions

- **Single file.** Keep the game in `index.html`. Do not split it into modules or add external dependencies or CDN links unless the user asks.
- **Plain JavaScript and CSS.** Match the existing style: `const`/`let`, arrow functions for callbacks, two-space indentation, single quotes, and terse one-line object literals for table data.
- **Tuning lives in tables.** Add new enemies, bosses, themes, or difficulty values as entries in the existing constant tables rather than as hard-coded numbers in logic.
- **Persist only through the `skyfighter.*` keys.** Wrap new `localStorage` access in the same pattern as the existing code.
- **Accessibility.** Respect `prefers-reduced-motion` (the `reducedMotion` flag gates screen shake, flash, and pulse). Announce important state changes through the `announce()` live region. Keep icon-only buttons labelled with a `title`, as the existing HUD buttons do.
- **Mobile.** Touch controls appear only on touch devices (`isTouchDevice`). Any new control needs a touch path as well as keyboard/mouse.
- **Keyboard controls** (for reference when changing input): arrows/WASD move, Space fires, X/Shift bomb, P/Esc pause.

## Making changes

- Read the relevant section of `index.html` before editing; the file is large and sections depend on shared state variables declared near the top of the IIFE.
- Verify each change by playing it in a browser. Check the touch layout if the change touches HUD or controls.
- Keep commits focused, with a clear message describing the gameplay or code change.
- Do not commit generated or temporary files.
