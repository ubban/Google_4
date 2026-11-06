# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo is **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs entirely in the browser. It is a single self-contained file:

- `index.html` (~1,670 lines) — all HTML, CSS (`<style>` block, ~lines 10–362), markup (canvas + HUD + overlays + touch controls), and JavaScript (one IIFE in a `<script>` block starting ~line 422).
- `README.md` — minimal placeholder.

There is no build step, no package manager, no bundler, no tests, and no external dependencies. Do not introduce any of these unless the user asks.

## Running and checking changes

- Open `index.html` directly in a browser, or serve the directory with any static server (e.g. `python3 -m http.server`).
- Verify gameplay changes in a real browser. There is no automated test suite.
- Keep the game working on both desktop (keyboard) and touch devices — the touch joystick and fire/bomb buttons are part of the core UI.

## Architecture

All game code lives inside one IIFE. Key pieces, roughly in file order:

- **Config constants**: `DIFFICULTIES` (easy/normal/hard tuning: spawn rate, enemy speed, shot chance, lives, bombs, boss HP multiplier), `KILLS_PER_STAGE`, `THEMES`, `ENEMY_TYPES`, `BOSSES`, `BOSS_PATTERNS`.
- **Persistence**: `localStorage` keys prefixed `skyfighter.` — `bestScore`, `muted`, `difficulty`, `reducedMotion`. Reads are not wrapped in try/catch; keep that in mind if you add more storage access.
- **Audio**: Web Audio API, created lazily via `ensureAudio()` (required because browsers gate audio on a user gesture). Sound effects are synthesized (`beep`, `noiseBurst`), not loaded from files.
- **Layout**: `resize()` scales the fixed `BASE_W × BASE_H` (480×720) logical canvas to the viewport.
- **Input**: keyboard state in `keys[e.code]`; Space or the fire button fires; `KeyP`/`Escape` pause; `KeyX`/Shift drop a bomb. Touch uses a joystick (`joystickStart/Move/End`) plus fire and bomb buttons. Mouse/pointer input goes through `canvasPointFromEvent`.
- **Game state**: `resetGame()` initializes state; `update(dt)` advances simulation; `draw()` renders; `loop(now)` drives both via `requestAnimationFrame`, clamping `dt` to 0.05s.
- **Bosses**: `spawnBoss`, `updateBoss`, `killBoss`, with attack patterns in `BOSS_PATTERNS` (radial, aimed spread, columns, spiral, cross, homing, double ring).
- **Accessibility**: `prefers-reduced-motion` is honored by default and can be toggled; `srLive` is an aria-live region used by `announce()`.

## Conventions

- Plain HTML/CSS/JS, ES2015+ features only. No frameworks, no transpilation.
- Keep everything in `index.html` unless the user asks to split files. Do not add new dependencies or CDN links.
- Preserve the `'use strict'` IIFE structure; don't leak globals.
- Use the existing naming style: `camelCase` for functions and variables, `UPPER_SNAKE` for constants, `skyfighter.*` for storage keys.
- Gameplay numbers belong in the config constants at the top of the script, not scattered as magic numbers.
- Keep changes minimal and scoped to the request. Don't reformat unrelated code.

## Git workflow

- Default branch work happens on the branch the session specifies (currently `claude/gifted-tesla-i31r4o`). Don't push to other branches without permission.
- Commit with clear, descriptive messages. Push with `git push -u origin <branch>`.
- Do not open a pull request unless the user explicitly asks for one.
