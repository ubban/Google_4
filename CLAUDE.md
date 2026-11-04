# CLAUDE.md

Guidance for Claude Code when working in this repository.

## What this repo is

A single-file, dependency-free HTML5 canvas game called **Sky Fighter**: a Raiden-style vertical shoot-'em-up. The whole game (markup, CSS, and JavaScript) lives in `index.html`. `README.md` is a placeholder and has no build or usage instructions.

Files:
- `index.html` (~1670 lines): the entire game. Three sections: `<style>` (~lines 10–362), the DOM markup including the overlay, HUD, and touch controls (~lines 363–421), and one IIFE `<script>` (~lines 422 onward).
- `README.md`: placeholder only.

There is no package.json, no build step, no bundler, no test suite, and no linter configured. Do not invent npm scripts or tooling. Do not add dependencies or CDN links without asking, since the project is intentionally self-contained.

## Running and verifying

- Open `index.html` directly in a browser, or serve the directory with any static server (e.g. `python3 -m http.server`).
- There are no automated tests. For behavioral changes, verify in a real browser (the pre-installed Playwright/Chromium is fine) and check the canvas, HUD, and overlay, not just the code.
- Test both desktop (keyboard/mouse) and touch layouts when changing input or layout code. Test at phone width, since the page uses `touch-action: none` and fixed-position layout.

## Architecture of the game script

All game state lives in closure variables inside the single IIFE. Key pieces, roughly in file order:

- **DOM references**: `const` lookups at the top of the IIFE (`canvas`, `ctx`, HUD elements, overlay, touch controls, etc.). Add new element lookups here.
- **Persistence keys**: `STORAGE_BEST`, `STORAGE_MUTE`, `STORAGE_DIFF`, `STORAGE_REDUCED` (all prefixed `skyfighter.`). Reads/writes go straight to `localStorage` without try/catch; keep that in mind if you add more.
- **Tuning tables**:
  - `DIFFICULTIES` (`easy` / `normal` / `hard`): spawn rate, enemy speed, shoot chance, starting lives and bombs, boss HP multiplier.
  - `KILLS_PER_STAGE` (18): kills needed to trigger a stage boss.
  - `THEMES`: sky gradient, cloud/sun colors, and names for six stage backgrounds (cycled by stage).
  - `ENEMY_TYPES`: `grunt`, `interceptor` (weaving), `tank` (4 HP). Each has a `minStage` gate.
  - `BOSSES`: six bosses (VANGUARD → TITAN), each with its own palette and `hpMul`.
- **Audio**: lazily created `AudioContext` (`ensureAudio`), with helpers `beep`, `noiseBurst`, and a `sfx` table. Audio is muted via `muted`/`STORAGE_MUTE`.
- **Responsive canvas**: `BASE_W = 480`, `BASE_H = 720`. `resize()` scales the canvas to the window, capped at 1.6×.
- **Input**:
  - Keyboard (`keys` map keyed by `e.code`): arrows/WASD move, Space fires, X or Shift bombs, P or Esc pauses.
  - Mouse: `mouseTarget` steers the ship; click fires.
  - Touch: a virtual joystick (`moveZone`), a FIRE button, and a BOMB button.
- **Simulation**: `update(dt)` (~line 1148) advances enemies, bullets, power-ups, collisions, scoring, and stage progression. `updateBoss(dt)` handles boss patterns and telegraphs.
- **Rendering**: `draw()` (~line 1520) renders everything each frame. Helpers include `drawPlane`, `drawBossShip`, and `drawClouds`.
- **Main loop**: `loop(now)` calls `update` and `draw` through `requestAnimationFrame`. Pause is handled by `togglePause()`.
- **Power-ups**: `type` is one of `life`, `bomb`, or `weapon` (weapon level has 3 tiers: Vulcan/Laser).
- **Scoring**: kill-chain multiplier (`mult`), best score persisted, "new best" banner on game over.
- **Accessibility**: `prefers-reduced-motion` is respected and can be toggled (`motionBtn`); `srLive` live region is used for announcements via `announce()`. Keep these working when adding visual effects (screen shake, flashes, vignette pulse).

## Conventions

- Keep everything in `index.html` unless the user asks to split it. Don't introduce a build step.
- Match the existing style: 2-space indentation, single quotes in JS, `const`/`let`, plain functions inside the IIFE, no classes or modules.
- Gameplay numbers belong in the tuning tables at the top of the script, not scattered as magic numbers. Prefer adding to those tables over hardcoding.
- The game runs a per-frame loop, so avoid heavy per-frame allocations or expensive effects in `update`/`draw`. `trackPerf` (~line 476) tracks frame times; check it when adding costly visuals.
- Keep the page single-viewport: `overflow: hidden`, `touch-action: none`, and `user-select: none` on `html, body` are intentional.
- Don't add external resources. The only asset is an inline SVG favicon.

## Git

- Develop on the branch assigned for the session (currently `claude/gifted-tesla-etu1q0`). Don't push to other branches without permission.
- Commit messages: short imperative summary line, then a body explaining the gameplay or code change where useful. The existing history uses this style (e.g. "Add Sky Fighter: Raiden-style vertical shoot-em-up game").
- Don't open a pull request unless explicitly asked.
