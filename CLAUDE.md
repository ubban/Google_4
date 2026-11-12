# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo is **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs in the browser. It is a single self-contained file:

- `index.html` — all markup, CSS (inline `<style>`), and game logic (one IIFE inside an inline `<script>`). No build step, no bundler, no dependencies.
- `README.md` — minimal placeholder.

There is no `package.json`, no test suite, and no linter configured. To run the game, open `index.html` in a browser (or serve the directory with any static server).

## Architecture of `index.html`

- **Canvas rendering:** a fixed logical resolution of `BASE_W = 480` × `BASE_H = 720`. `resize()` scales the canvas CSS size to fit the viewport while keeping the internal resolution constant. Game coordinates always use the logical size.
- **Game loop:** `update(dt)` then `draw()`, driven by `requestAnimationFrame` in `loop(now)`. Frame time is clamped, and `trackPerf` adapts a `quality` level (0–2) when the frame rate drops.
- **Config tables** (near the top of the script): `DIFFICULTIES` (easy/normal/hard tuning for spawn rate, enemy speed, shot chance, lives, bombs, boss HP), `THEMES` (per-stage sky palettes, cycled by stage), `ENEMY_TYPES` (grunt, interceptor, tank), `BOSSES` (six bosses, one per stage cycle), `KILLS_PER_STAGE = 18` (kills needed before the boss appears).
- **Weapons:** `vulcan` (default) and `laser`, toggled through a power-up. Fire delay is 0.15 s for vulcan and 0.1 s for laser. Weapon level is tracked on `player.level`.
- **Bosses:** `spawnBoss`, `updateBoss`, and attack patterns (`fireRadial`, `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`, `fireCross`, `fireHomingSeeded`, `fireDoubleRing`). The boss health bar is driven by `bossBarWrap` / `bossBarFill`.
- **Audio:** a Web Audio synth (`ensureAudio`, `beep`, `noiseBurst`, `sfx`). Audio is created lazily on the first user gesture, so keep it that way. Mute state persists.
- **Input:** keyboard (arrows/WASD to move, Space to fire, X / Shift for bomb, P / Esc to pause), mouse (hover-to-steer on the canvas, mousedown fires), and touch (a joystick in `moveZone`, plus `fireBtn` and `bombBtn`).
- **HUD and overlay:** DOM elements (`#hud`, `#hearts`, `#bombs`, `#overlay`, `#stageBanner`, `#comboPop`) sit above the canvas. `announce()` writes to a live region for screen readers.
- **Persistence:** `localStorage` keys are prefixed `skyfighter.` (`bestScore`, `muted`, `difficulty`, `reducedMotion`). Every read and write is a plain call, so keep them defensive if you add more.

## Conventions

- Keep the game self-contained in `index.html`. Do not introduce a build system or external dependencies unless the user asks.
- Keep the IIFE wrapper and `'use strict'`. Game state lives in closure variables, not on `window`.
- Use the logical `BASE_W` / `BASE_H` coordinate space for any new game objects. Do not rely on the CSS size of the canvas.
- Respect `reducedMotion` when adding screen shake, flashes, or other motion effects. Respect `muted` for every sound.
- Keep the mobile layout working: the page is `overflow: hidden`, uses `touch-action: none`, and must stay usable at phone widths.
- Preserve accessibility hooks (`aria-*` attributes, the `srLive` region) when editing overlay or HUD markup.

## Working in this repo

- Make small, focused edits with the existing style (2-space indentation, single quotes in JS, `const`/`let`).
- Verify changes by loading the page in a browser and playing a few stages, especially any change touching the loop, bosses, or input. There are no automated tests to run.
- Commit on the development branch named in the session instructions. Do not push to other branches without permission.
