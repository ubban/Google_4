# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo is a small, dependency-free browser game: **Sky Fighter**, a Raiden-style vertical shoot-'em-up rendered on an HTML5 `<canvas>`.

The entire game lives in one file, `index.html` (~1,670 lines): markup, CSS in a `<style>` block, and JavaScript in a `<script>` block wrapped in an IIFE with `'use strict'`. `README.md` is a placeholder and carries no setup instructions.

There is no build step, no package manager, no bundler, no test suite, and no linter configuration. Do not add any of these unless asked.

## Running and verifying

- Open `index.html` directly in a browser, or serve the directory with any static server (e.g. `python3 -m http.server`).
- There are no automated tests. Verify changes by playing the game in a browser. For UI or gameplay changes, check desktop (mouse + keyboard) and touch layouts, since touch controls are shown only on touch-capable devices.
- Syntax-check the inline script after edits if convenient, e.g. extract the `<script>` block and run `node --check`.

## Code layout (`index.html`)

Top to bottom:

1. **`<head>`**: meta tags, an inline SVG favicon, and the stylesheet. Colors and HUD styling are plain CSS.
2. **`<body>`**: the `#container`, the `<canvas id="gameCanvas">`, the HUD (`#hud`, score, weapon, hearts, bombs, boss bar, stage banner), the start/pause/game-over overlay (`#overlay`), touch controls (`#touchControls`), and a screen-reader live region (`#srLive`).
3. **Constants** near the top of the script:
   - `STORAGE_*` keys (all prefixed `skyfighter.`), used for localStorage persistence.
   - `DIFFICULTIES` (easy / normal / hard): spawn rates, enemy speed and fire chance, starting lives and bombs, boss HP multiplier.
   - `KILLS_PER_STAGE`, `THEMES` (per-stage sky palettes), `ENEMY_TYPES` (grunt, interceptor, tank, each with a `minStage`), `BOSSES` (rotated by stage).
   - `BASE_W = 480`, `BASE_H = 720`: the logical canvas size. `resize()` scales the canvas to fit the window, capped at 1.6x.
4. **Subsystems**: adaptive quality (`trackPerf` drops visual quality when frame time is high), a Web Audio `sfx` object, input handling (keyboard, mouse, joystick/touch), and the main `loop` driven by `requestAnimationFrame`.
5. **Game state and flow**: `startGame`, stage progression (kill count per stage, then a boss fight), the boss, bombs, power-ups, scoring with a kill-chain multiplier, and game over.

Keep to this structure. New constants belong with the existing ones, new tunables belong in `DIFFICULTIES` if they should vary by difficulty, and new persisted values should get a `skyfighter.*` storage key.

## Conventions

- **Single file, no dependencies.** Do not introduce external scripts, stylesheets, or fonts. Keep everything inline in `index.html`.
- **Stay in the IIFE.** Do not expose new globals; the game intentionally keeps state private.
- **Keep the code style.** Two-space indentation, single quotes in JS, `const`/`let` (no `var`), and the `// ---------- Section ----------` comment banners.
- **Canvas coordinates.** Game logic works in the fixed 480×720 space. Do not depend on the CSS size of the canvas.
- **Respect the accessibility hooks.** Keep the `reducedMotion` toggle working (`body.reduced-motion` disables screen-shake and similar effects), keep announcements going through `announce()` so they reach `#srLive`, and keep the mute toggle effective for all `sfx` calls.
- **Persistence is best-effort.** localStorage reads and writes can fail (private browsing, blocked storage). Keep the existing `Number(... ) || 0` style fallbacks rather than assuming storage exists.
- **Touch and mobile.** `touch-action: none` and `user-select: none` are set on purpose to stop page gestures from interfering with play. Preserve safe-area insets (`env(safe-area-inset-*)`) and the `viewport-fit=cover` meta tag.

## Git and workflow

- Default working branch for this session: `claude/gifted-tesla-rqzmdd`.
- Commit with clear messages that describe the gameplay or UI change.
- Do not open pull requests unless explicitly asked.
