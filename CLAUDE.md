# CLAUDE.md

Guidance for Claude Code when working in this repository.

## What this repo is

A small, dependency-free browser game: **Sky Fighter**, a Raiden-style vertical shoot-'em-up. The entire game lives in one file, `index.html`:

- Markup and HTML overlays (menus, HUD, touch controls) at the top of the file.
- All CSS inline in a `<style>` block (~lines 10–362).
- All JavaScript inline in one IIFE `<script>` block (starts ~line 422). Nothing is imported or exported.

`README.md` is a placeholder and carries no project documentation.

## Running and testing

- No build step, package manager, or test suite. There is no `package.json`.
- To run: open `index.html` directly in a browser, or serve the directory with any static server (e.g. `python3 -m http.server`).
- There are no automated tests. Verify changes by playing the game in a browser: start a run, exercise the change, and check the console for errors. For UI or visual changes, take a screenshot.

## Code layout inside the script

The script is organized by section comments. Key pieces:

- **DOM refs and storage keys**: element lookups at the top; `localStorage` keys prefixed `skyfighter.` (`bestScore`, `muted`, `difficulty`, `reducedMotion`).
- **Adaptive performance** (`trackPerf`): samples frame times and steps a `quality` level (0 full / 1 medium / 2 low) up or down.
- **Tuning tables**: `DIFFICULTIES` (spawn rates, enemy speed and fire chance, lives, bombs, boss HP multiplier), `KILLS_PER_STAGE`, `THEMES` (sky palettes per stage), `ENEMY_TYPES` (grunt, interceptor, tank), `BOSSES` (six bosses with names, colors, HP multiplier), and `BOSS_PATTERNS` (bullet-pattern functions).
- **Audio**: Web Audio helpers (`ensureAudio`, `beep`, `noiseBurst`). Audio is created lazily on the first user gesture, so keep calls to `ensureAudio()` on input handlers.
- **Input**: keyboard handlers on `window`, touch joystick and buttons, and pointer mapping via `canvasPointFromEvent`.
- **Game state and loop**: `resetGame`, `update(dt)` for logic, `draw()` for rendering, and `loop(now)` driving both via `requestAnimationFrame`. `dt` is clamped to 0.05 s.
- **Rendering helpers**: `drawPlane`, `drawBossShip`, `drawClouds`, plus screen shake and flash effects.

Controls (for reference when editing input code):
- Move: arrow keys or WASD. Fire: Space. Bomb: X or Shift. Pause: P or Escape.
- Touch: left-side joystick, right-side fire and bomb buttons.

## Conventions

- Keep everything in `index.html` unless the user asks to split it. Do not introduce a build step or external dependencies without asking.
- Keep the IIFE and `'use strict'`. Do not add globals.
- Gameplay numbers belong in the tuning tables at the top of the script, not hard-coded in logic.
- Respect the reduced-motion setting: effects like screen shake should check `reducedMotion`.
- Wrap all `localStorage` access in a way that tolerates it being unavailable (private windows, blocked storage). The current code calls it directly; new code should not make that worse.
- Keep the layout working at phone widths, since touch controls are a first-class input.
- Do not commit, push, or open a PR unless the user asks.

## Git

- Default development branch for this session: `claude/gifted-tesla-rzkotw`.
- Commit messages should be short and imperative, describing the gameplay or code change.
