# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo contains **Sky Fighter**, a Raiden-style vertical shoot-em-up built as a single-page HTML5 canvas game.

- `index.html` — the entire game: markup, CSS (in `<style>`), and JavaScript (in one `<script>` IIFE). There is no build step, no bundler, and no package manager.
- `README.md` — minimal project title only.

To run it, open `index.html` in a browser (or serve the directory with any static server). Nothing needs to be installed.

## Layout of `index.html`

- **Lines ~1–360:** `<head>` metadata and the `<style>` block (HUD pills, overlay card, touch controls, reduced-motion styles).
- **Lines ~361–421:** DOM structure. The canvas `#gameCanvas` sits in `#container`, with HUD, boss bar, overlay, touch controls, and a screen-reader live region (`#srLive`).
- **Lines ~422 onward:** the game script, wrapped in `(function () { 'use strict'; ... })()`. Sections, roughly in order:
  - DOM lookups and `STORAGE_*` keys
  - Adaptive performance (`quality` levels 0–2, driven by frame times)
  - Tuning tables: `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES`, `ENEMY_TYPES`, `BOSSES`
  - Audio (`ensureAudio`, `beep`, `noiseBurst`, `sfx`)
  - HUD and effect helpers (hearts, bombs, shake, flashes, combo and stage banners)
  - Input: keyboard, mouse, and touch joystick/buttons
  - Gameplay: `resetGame`, `spawnEnemy`, `spawnBoss`, `fireWeapon`, `useBomb`, `update`, `applyPowerUp`
  - Boss patterns: `BOSS_PATTERNS` and the `fire*` helpers, plus `updateBoss` and `killBoss`
  - Rendering: `drawPlane`, `drawBossShip`, `drawClouds`, `draw`
  - Main loop: `loop(now)` calls `trackPerf`, then `update`, then `draw`, via `requestAnimationFrame`

## Conventions

- **Keep it a single file.** The project is deliberately dependency-free. Do not add a bundler, framework, or npm dependencies unless the user asks for them.
- **Match the existing style.** Use `const`/`let`, arrow functions for callbacks, two-space indentation, and single quotes. Keep the section banners in the form `// ---------- Name ----------`.
- **Balance tuning changes.** Difficulty numbers live in `DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, and `BOSS_PATTERNS`. Change values there, not inline in gameplay code.
- **Persist state through the existing keys.** Use the `skyfighter.*` `localStorage` keys defined near the top of the script. Wrap new storage access so the game still works if storage is unavailable.
- **Preserve accessibility hooks.** Announce important events through `announce()`, which feeds `#srLive`. Respect `reducedMotion`, which gates screen shake, flashes, and pulses.
- **Keep input parity.** Mouse, keyboard, and touch controls each have their own handlers. A change to one input path should be checked against the others.
- **Mobile layout.** The game scales to a `480×720` logical canvas (`BASE_W`/`BASE_H`) through `resize()`. Check the touch controls at phone width when you change HUD or overlay layout.

## Testing and verification

There is no automated test suite or linter in the repo. Verify changes by:

1. Opening `index.html` in a browser and playing through at least one stage, including a boss fight if you touched boss or weapon code.
2. Checking all three difficulty levels if you changed tuning values.
3. Checking the pause (`P`/`Esc`), mute, reduced-motion, and restart flows if you touched the UI.
4. Checking touch controls in a mobile-sized viewport if you touched input or layout.
5. Running a quick syntax check on the inline script if you want a fast signal, for example by extracting the `<script>` body and running `node --check` on it.

## Git workflow

- Develop on the branch assigned for the session. Do not push to other branches without explicit permission.
- Write clear, descriptive commit messages. The existing history uses an imperative subject line followed by a bulleted body.
- Do not open a pull request unless the user explicitly asks for one.
