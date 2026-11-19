# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo contains **Sky Fighter**, a single-file, browser-based vertical shoot-em-up (Raiden-style) game.

- `index.html` — the entire game: markup, CSS (`<style>`, ~lines 10–362), canvas element, and JavaScript (`<script>`, ~lines 422–1667).
- `README.md` — minimal placeholder. Keep it accurate if you change project-level information.

There is no build step, no package manager, no bundler, and no test suite. Nothing needs to be installed.

## Running the game

Open `index.html` directly in a browser, or serve the directory with any static server, e.g.:

```bash
python3 -m http.server 8000   # then visit http://localhost:8000/index.html
```

## Architecture notes

- The game logic lives in a single IIFE (`(function () { 'use strict'; ... })();`). Nothing is exported to `window`; keep new code inside that scope unless there's a reason not to.
- Rendering is 2D canvas (`gameCanvas`). The logical play field is `BASE_W = 480`, `BASE_H = 720`, scaled by `resize()`.
- The main loop is `requestAnimationFrame` → `loop(now)`, which clamps `dt` to 0.05s and calls `update(dt)` then `draw()` while `running && !paused`.
- Key game data is declared as constants near the top of the script:
  - `DIFFICULTIES` (easy / normal / hard): spawn rates, enemy speed, shot chance, lives, bombs, boss HP multiplier.
  - `KILLS_PER_STAGE` (18): kills needed before a boss spawns.
  - `THEMES`: six background palettes that cycle with stages.
  - `ENEMY_TYPES`: grunt, interceptor, tank.
  - `BOSSES`: six bosses, each with colors, name, and HP multiplier; attack patterns live in `BOSS_PATTERNS`.
- Audio is generated with the Web Audio API (`ensureAudio`, `beep`, `noiseBurst`). No audio files are loaded.
- Persistent settings use `localStorage` with the `skyfighter.*` key prefix: best score, mute, difficulty, and reduced motion. Every access should tolerate storage being unavailable or throwing.
- Accessibility: `prefers-reduced-motion` seeds the reduced-motion setting; an `aria-live` region (`srLive`) is used via `announce()` for screen-reader updates.

## Controls (for reference when editing input code)

- Keyboard: Arrow keys or WASD move (tracked in `keys[e.code]`, read in `update()`); Space fires; X or Shift uses a bomb; P or Escape pauses.
- Mouse: moving over the canvas steers the ship (`mousemove` → `canvasPointFromEvent`); clicking fires.
- Touch: on touch devices, `touchControls` is enabled with a virtual joystick (`moveZone`), a fire button (`fireBtn`), and a bomb button (`bombBtn`).

## Conventions

- Keep the game self-contained in `index.html` unless the user asks for a split. Don't add a build step or dependencies for small changes.
- Use `const`/`let`, the existing 2-space indentation, and single quotes to match the surrounding code.
- Don't add external CDN scripts or fonts without asking; the game currently has no external dependencies.
- When adding new enemies, bosses, or themes, extend the existing tables (`ENEMY_TYPES`, `BOSSES`, `THEMES`, `BOSS_PATTERNS`) rather than adding one-off branches.
- Preserve the `skyfighter.*` localStorage key names; changing them would reset players' saved settings and best score.

## Verifying changes

There are no automated tests. After changing game logic or markup:

1. Open `index.html` in a browser (or serve it locally) and check the browser console for errors.
2. Start a game, play through at least one stage transition, and confirm the boss fight starts after `KILLS_PER_STAGE` kills.
3. Check pause (P/Escape), bomb use, game over, and that the best score persists after reload.
4. If touch behavior changed, test it in a touch-emulated browser viewport.
