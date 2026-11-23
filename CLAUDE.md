# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a small static web project. Its only real content is **Sky Fighter**, a
Raiden-style vertical shoot-em-up written as a single self-contained HTML5 canvas page.

| File        | Purpose                                                        |
|-------------|----------------------------------------------------------------|
| `index.html`| The whole game: markup (~lines 1–420), CSS (inline `<style>`), and JavaScript (inline `<script>`, starts ~line 422) |
| `README.md` | Minimal placeholder (title only)                              |

There is no build step, no package manager, no test suite, and no external assets.
Audio is synthesized with WebAudio, and graphics are drawn procedurally on the canvas.

## Running and verifying

- Open `index.html` directly in a browser, or serve the directory with any static server
  (e.g. `python3 -m http.server`).
- There are no automated tests or linters configured. Verify changes by loading the page
  and playing, or with a headless browser (Playwright/Chromium is preinstalled in the
  remote environment).

## Code structure (inside `index.html`)

The script is one IIFE with `'use strict'`. Major sections, in rough order:

1. **DOM references** – `getElementById` lookups for HUD, overlay, and touch controls.
2. **Storage keys** – `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`,
   `skyfighter.reducedMotion` in `localStorage`.
3. **Accessibility/perf helpers** – `announce()` (screen-reader live region) and `trackPerf()`.
4. **Settings** – `DIFFICULTIES` table (per-difficulty tuning, read as `d = DIFFICULTIES[difficulty]`),
   `KILLS_PER_STAGE` (kills needed before a boss warning/fight).
5. **Audio** – `ensureAudio()`, `beep()`, `noiseBurst()`, and the `sfx` helpers. Audio is
   created lazily on the first user gesture; respect the mute toggle.
6. **HUD/effects** – hearts, bombs, combo pop, stage banner, screen shake, flash.
7. **Input** – keyboard (`keys[e.code]`), mouse on canvas, touch joystick (`moveZone`),
   fire and bomb buttons. Keyboard, mouse, and touch all feed the same player state.
8. **Game state** – `resetGame()`, `update(dt)`, `spawnEnemy()`, `spawnBoss()`,
   `maybeSpawnPowerUp()`, `addScore()`, combo chain (`breakChain()`), `useBomb()`.
9. **Weapons and boss patterns** – `fireWeapon()`, `applyPowerUp()`, and boss bullet
   patterns (`fireRadial`, `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`,
   `fireCross`, `fireHomingSeeded`, `fireDoubleRing`), driven by `updateBoss()`.
10. **Rendering** – `draw()` and helpers (`drawPlane`, `drawBossShip`, `drawClouds`).
    Drawing is pure canvas 2D with no sprites.
11. **Main loop** – `loop(now)` calls `update(dt)` then `draw()` via `requestAnimationFrame`.
    Pausing is handled by `togglePause()`.

## Conventions

- **Keep it a single file.** Don't split out modules, add a bundler, or add external
  dependencies or CDN scripts unless the user asks for it. Everything stays inline.
- **No external assets.** Sounds and graphics are generated in code. Don't add image or
  audio files without an explicit request.
- **Preserve the IIFE and `'use strict'`.** Don't leak globals; keep new state inside the closure.
- **Use `DIFFICULTIES` for tuning values** rather than hard-coding numbers in the game
  logic, so all three difficulty levels stay consistent.
- **Use the existing storage keys** and wrap `localStorage` access in a way that still works
  if storage is unavailable (private mode, blocked site data).
- **Respect the reduced-motion and mute settings** when adding new visual or audio effects.
- **Keep input parity.** Any new action needs keyboard, mouse, and touch paths, and the
  on-screen hint (`#controlsHint`) should be updated to match.
- **Style follows the existing inline CSS** in the `<style>` block. Reuse existing selectors
  and match the current look before adding new styling.

## Git

- Development branch for this session: `claude/gifted-tesla-w7kp32`. Push there with
  `git push -u origin <branch>`.
- Commit messages should describe the gameplay or UI change in plain terms. Recent history
  uses a short summary line followed by a bulleted body.
- Don't open pull requests unless explicitly asked.
