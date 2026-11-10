# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Google_4 is a small, dependency-free browser game: **Sky Fighter**, a Raiden-style vertical shoot-'em-up. The entire game is a single file, `index.html`, containing inline CSS and one inline `<script>` (an IIFE). There is no build step, no package manager, no test suite, and no external assets. `README.md` is a placeholder.

To run it, open `index.html` in a browser (or serve the directory with any static file server).

## Repository layout

- `index.html` (~1,670 lines): the complete game.
  - Lines ~10–359: inline `<style>` (HUD, overlays, touch controls, vignette/flash effects, responsive sizing).
  - Lines ~363–421: markup (canvas `#gameCanvas`, HUD, start/pause overlays, touch controls, difficulty buttons, live region).
  - Lines ~422–1667: the game script, all inside one IIFE with `'use strict'`.
- `README.md`: placeholder title only.

## Architecture of the game script

Everything is a closure-scoped function; there are no modules or globals beyond the DOM.

- **Setup and persistence.** DOM element references are cached at the top. `localStorage` keys use the `skyfighter.*` prefix: `bestScore`, `muted`, `difficulty`, `reducedMotion`. Any new persisted setting should follow this prefix and be read defensively (values come back as strings or `null`).
- **Config tables** (top of the script, easy to tune):
  - `DIFFICULTIES` (`easy` / `normal` / `hard`): spawn cadence, enemy speed, enemy shoot chance, starting lives and bombs, boss HP multiplier.
  - `KILLS_PER_STAGE` (18): kills required before a stage boss is spawned.
  - `THEMES` (6 sky palettes, cycled by stage).
  - `ENEMY_TYPES` (`grunt`, `interceptor`, `tank`), gated by `minStage`.
  - `BOSSES` (6 boss designs, cycled by stage; HP is `(70 + stage * 35) * difficulty.bossHpMul * design.hpMul`).
- **Game loop.** `loop(now)` drives `update(dt)` and `draw()` via `requestAnimationFrame`. `trackPerf` adapts a `quality` level (0–2) from the average frame time, which drives visual detail (clouds, effects).
- **Input.** Keyboard (Arrows or WASD to move, Space to fire, `X`/`Shift` for bomb, `P`/`Esc` to pause; state is tracked in the `keys` map by `e.code`), mouse on the canvas, and touch (virtual joystick in `#moveZone`, fire and bomb buttons). Touch and mouse handlers call `preventDefault` where needed to stop page scrolling.
- **Weapons and power-ups.** `player.weapon` is `vulcan` or `laser`, with `player.level` 1–3. `applyPowerUp` handles `life`, `bomb`, and `weapon` pickups. Once a weapon is at level 3, the next weapon pickup swaps to the other weapon at level 1.
- **Boss patterns.** `fireRadial`, `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`, `fireCross`, `fireHomingSeeded`, and `fireDoubleRing` are the boss attack primitives, used by `updateBoss`.
- **Rendering.** All drawing goes to a single 2D canvas context. Logical size is `BASE_W` x `BASE_H` (480 x 720); `resize()` scales it to fit the viewport. Ship and boss shapes are drawn procedurally (`drawPlane`, `drawBossShip`); there are no image or sprite files.
- **Feedback.** Web Audio sound effects via `beep` / `noiseBurst` (muted by `muted`, initialised lazily by `ensureAudio` after a user gesture). Screen shake, hit flash, vignette, combo pops, and stage banners. A `reducedMotion` setting disables the motion-heavy effects.
- **Accessibility.** The `#srLive` ARIA live region is updated through `announce()` for game events (stage change, boss defeat, game over). Keep announcements short and use `announce()` for any new major game state.

## Conventions

- Keep the game self-contained in `index.html`. Do not add a bundler, framework, or npm dependency unless the user asks for it.
- Use plain ES2015+ JavaScript inside the existing IIFE, matching the current style: `const`/`let`, arrow functions, two-space indentation, single quotes in JS.
- Tune gameplay through the existing config tables rather than hard-coding numbers in the update loop.
- Preserve mobile behaviour: the page is `overflow: hidden` with `touch-action: none`, and the layout must still work at phone widths with safe-area insets.
- When changing the HUD or overlays, check both the desktop and narrow (phone) layouts.

## Working in this repo

- There are no automated tests or linters configured. Verify changes by opening `index.html` in a browser and playing through at least one stage, and by checking the browser console for errors.
- Git: work on the branch the session specifies. Commit with clear messages describing the gameplay or code change.
- Do not add a PR unless explicitly asked.
