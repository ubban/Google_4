# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo is a small, dependency-free browser game, **Sky Fighter**: a Raiden-style vertical shoot-'em-up.

- `index.html` — the entire game: inline CSS, markup, and one inline `<script>` (no build step, no external assets, no frameworks).
- `README.md` — minimal project title only.

There is no package.json, test suite, linter, or build tooling. To run the game, open `index.html` in a browser (or serve the directory with any static server).

## Code layout (`index.html`)

- **Styles** — inline `<style>` in `<head>`. HUD, overlay, touch controls, vignette/flash effects, and keyframes live here. Colors are hard-coded hex/rgba values.
- **Markup** — `#container` wraps `#gameCanvas` plus the HUD (`#hud`, `#hearts`, `#bombs`, `#weaponStat`, `#multStat`), overlay/menu (`#overlay`, `#startBtn`, `#overlayTitle`), boss bar (`#bossBarWrap`), touch controls (`#touchControls`, `#moveZone`, `#fireBtn`, `#bombBtn`), and an ARIA live region (`#srLive`).
- **Script** — one `(function () { 'use strict'; ... })()` block starting near line 421. Key sections, in order:
  - DOM references (`const xxxEl = document.getElementById(...)`).
  - `STORAGE_*` keys — persisted under the `skyfighter.*` namespace in `localStorage` (best score, mute, difficulty, reduced motion).
  - Perf tracking (`trackPerf`, `quality` 0–2) — lowers effect quality when frame times degrade.
  - Tuning tables: `DIFFICULTIES` (easy/normal/hard), `KILLS_PER_STAGE`, `THEMES` (per-stage sky palettes), `ENEMY_TYPES` (grunt, interceptor, tank), `BOSSES` (six bosses, selected by `(stage - 1) % BOSSES.length`).
  - Audio — lazily created `AudioContext` via `ensureAudio()`; sound effects are synthesized with `beep()` / `noiseBurst()` and the `sfx` table. No audio files.
  - Input — keyboard (`keys` map keyed by `e.code`), touch joystick + fire/bomb buttons, and mouse fire button.
  - Game loop — `requestAnimationFrame(loop)`; `update` and draw logic for player, bullets, enemies, bosses, bombs, particles, and HUD.

## Conventions

- **Keep it a single file.** Do not introduce a bundler, framework, or external dependency unless the user explicitly asks. New CSS and JS go into the existing `<style>` and `<script>` blocks.
- **Match the existing style.** 2-space indentation, single quotes in JS, `const`/`let` (no `var`), camelCase identifiers, and `SCREAMING_SNAKE_CASE` for tuning constants and storage keys.
- **Tuning lives in the data tables.** Prefer adjusting values in `DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, and `THEMES` over hard-coding numbers in logic.
- **Mobile and accessibility matter.** The page supports touch (joystick + buttons), safe-area insets, `prefers-reduced-motion`, and screen-reader announcements via `announce()`. Preserve these when changing UI or effects.
- **Persisted state** goes through the `STORAGE_*` constants and should be wrapped so the game still works if `localStorage` is unavailable.

## Controls (for reference when testing)

| Action | Keyboard | Touch / mouse |
| --- | --- | --- |
| Move | Arrow keys or WASD | Virtual joystick |
| Fire | Space (hold) | Fire button (hold) / mouse button on fire button |
| Smart bomb | X or Shift | Bomb button |
| Pause | P or Esc | Pause button |

## Verifying changes

Since there is no automated test setup:

1. Open `index.html` in a desktop browser and check the console for errors.
2. Play through at least one stage transition (18 kills) and a boss fight to exercise the bosses, themes, and HUD.
3. Check the touch layout at phone width (or with device emulation) for overflow and control placement.
4. Confirm best score, mute, difficulty, and reduced-motion settings persist across reloads.

## Git

- Develop on the branch assigned for the task; do not push to other branches without permission.
- Write clear, descriptive commit messages in the imperative mood.
