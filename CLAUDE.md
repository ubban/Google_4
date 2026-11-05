# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo contains **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs as a single-page HTML5 canvas game.

- `index.html` — the entire game: markup, CSS (inline `<style>`), and JavaScript (inline `<script>`, wrapped in one IIFE).
- `README.md` — minimal placeholder.

There is no build system, package manager, bundler, framework, or test suite. Do not add one unless asked. To run the game, open `index.html` in a browser (or serve the directory with any static file server).

## Architecture of `index.html`

Roughly in file order:

1. **Head / CSS** (~lines 1–420): viewport meta for mobile (no zoom), inline SVG favicon, dark color scheme, HUD pills (`#hud`, `.stat`), heart/bomb indicators, vignette and flash overlays, the title/overlay card, and touch controls.
2. **DOM markup** (~lines 400–420): overlay card (`#overlay`, `#startBtn`, `#controlsHint`), HUD elements, touch joystick (`#moveZone`, `#joystickBase`, `#joystickStick`), `#fireBtn`, `#bombBtn`, the canvas `#gameCanvas`, and a polite live region `#srLive` for screen readers.
3. **Script** (`<script>` starting ~line 422): one IIFE with `'use strict'`. Sections, in order:
   - **Element lookups and storage keys** — `STORAGE_BEST`, `STORAGE_MUTE`, `STORAGE_DIFF`, `STORAGE_REDUCED` (all prefixed `skyfighter.`).
   - **Accessibility/perf helpers** — `announce()` (live region), `trackPerf()` (rolling frame-time average that adjusts `quality` from 0–2).
   - **Tuning data** — `DIFFICULTIES` (easy/normal/hard: spawn rates, enemy speed, shoot chance, lives, bombs, boss HP multiplier), `KILLS_PER_STAGE`, `THEMES` (per-stage sky palettes), `ENEMY_TYPES` (grunt, interceptor, tank), `BOSSES` (six boss designs, cycled by stage), `BOSS_PATTERNS`.
   - **Audio** — lazily created via `ensureAudio()`; `beep()` and `noiseBurst()` synthesize all sound with WebAudio (no audio files). `muted` persists.
   - **HUD rendering** — `renderHearts`, `renderBombs`, `showComboPop`, `showStageBanner`, `screenShake`, `flashHit`, `flashBomb`.
   - **Settings** — mute, difficulty, and reduced-motion toggles, each persisted to `localStorage`.
   - **Input** — keyboard (`keys` map, plus `Space`, `KeyP`/`Escape` pause, `KeyX`/`Shift` bomb), mouse on the canvas, and touch joystick/buttons. Mouse and touch both route into the same `firing` / movement state.
   - **Game state and loop** — `resetGame()`, `update(dt)` (per-frame simulation), `draw*` helpers (`drawPlane`, `drawBossShip`), spawners (`spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`), weapons (`fireWeapon`, `applyPowerUp`), boss attack patterns (`fireRadial`, `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`, `fireCross`, `fireHomingSeeded`, `fireDoubleRing`), and `updateBoss`/`killBoss`.
   - **Lifecycle** — `loseLife()`, `endGame()` (saves best score), `togglePause()`.

Key game rules to keep in mind when changing behavior:

- Scoring uses a kill-chain multiplier (`addScore`, `breakChain`).
- Bombs (`useBomb`) clear the screen and damage bosses; they are limited per difficulty.
- Weapons have 3 levels (Vulcan/Laser); power-ups raise level, grant bombs, or extra lives.
- Each stage ends after `KILLS_PER_STAGE` kills and triggers a boss fight; stage themes and boss designs advance with the stage number.

## Conventions

- **Single file.** Keep the game in `index.html` unless the user asks for a split. Do not introduce external dependencies, CDNs, or npm packages.
- **No frameworks.** Plain DOM APIs and `<canvas>` 2D context only.
- **Keep the existing style:** 2-space indentation, single quotes in JS, `const`/`let`, arrow functions for callbacks, and section comments in the form `// ---------- Name ----------`.
- **Persistence keys** must stay under the `skyfighter.` prefix. Wrap new `localStorage` reads/writes so the game still works if storage is unavailable or blocked.
- **Accessibility and mobile:** respect `prefers-reduced-motion` (via the `reducedMotion` setting), keep touch targets usable, and keep the `touch-action: none` / `user-select: none` behavior on the game surface. Announce important state changes through `announce()`.
- **Performance:** the game loop is per-frame; avoid allocations and DOM writes in hot paths when a cached value or a `quality`-gated effect will do.

## Working in this repo

- Work on the designated feature branch (currently `claude/gifted-tesla-04ityb`). Do not push to other branches without explicit permission.
- Commit with clear, descriptive messages. Push with `git push -u origin <branch>`.
- There are no automated tests or linters. Verify changes by opening `index.html` in a browser (or a headless Chromium) and playing through the affected path: start, fire, bomb, pause, stage transition, boss fight, and game over. Check mobile layout when touching HUD or touch controls.
- Do not create pull requests unless explicitly asked.
