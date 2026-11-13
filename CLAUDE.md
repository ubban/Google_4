# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a single-page HTML5 canvas game, **Sky Fighter**: a Raiden-style vertical shoot-'em-up. The entire app lives in `index.html` (one `<style>` block, one inline `<script>` block, ~1670 lines). `README.md` is a placeholder.

There is no build step, package manager, bundler, framework, or test suite. Do not add one unless asked.

## Running and verifying

- Open `index.html` directly in a browser, or serve the directory with any static server (e.g. `python3 -m http.server`).
- There are no automated tests or lint scripts. Verify changes by loading the page in a browser and playing through the affected path. For headless checks in this environment, Playwright's Chromium is pre-installed; don't reinstall it.
- Check the browser console for errors after any change to the script.

## Code layout (inside `index.html`)

- **Storage keys** (`STORAGE_*`, ~line 463): `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion`. Reads and writes go through `localStorage`; keep these key names stable so saved player data survives.
- **Tuning constants** (~lines 485–515):
  - `DIFFICULTIES`: `easy` / `normal` / `hard`, with spawn timing, enemy speed, shoot chance, starting lives and bombs, and boss HP multiplier.
  - `KILLS_PER_STAGE` (18): kills needed to advance a stage and trigger the boss.
  - `THEMES`: six sky palettes, cycled per stage.
  - `ENEMY_TYPES`: `grunt`, `interceptor` (weaving), `tank` (4 HP, appears from stage 2).
  - `BOSSES`: six bosses, one per stage, each with its own colors and HP multiplier. Attack patterns are in `BOSS_PATTERNS` (~line 1372).
- **Audio** (`ensureAudio`, ~line 545): a lazily created `AudioContext`, started on the first user gesture. Sound effects are gated by `muted`.
- **Adaptive quality** (`trackPerf`, ~line 478): lowers the render quality level when frame time stays high, and raises it again when frames are fast.
- **Input** (~lines 769–860): keyboard listeners, canvas mouse steering and click-to-fire (`mousemove` / `mousedown` on the canvas), and a touch joystick plus FIRE and BOMB buttons. The touch controls overlay is shown only when `isTouchDevice` is true (~line 778).
- **Game loop**: `update(dt)` (~line 1148) advances state, `draw()` (~line 1520) renders, and `loop(now)` (~line 1640) drives both via `requestAnimationFrame`.
- **Pause**: `togglePause()` (~line 1493), bound to `P` and `Esc`.

## Controls (for testing)

- Move: mouse, or Arrow keys / WASD. Fire: hold Space or click. Bomb: `X` or Shift.
- Pause: `P` or `Esc`. Mute: the speaker button.
- Mobile: drag left to fly, tap FIRE, tap BOMB.

## Conventions

- Keep everything in the single file unless the user asks for a split. Do not introduce external dependencies or CDN links.
- Keep the visual style (pill-style HUD, glassmorphic overlay, shaded planes) consistent with the existing code.
- Preserve mobile behavior: `touch-action: none`, safe-area insets, and the viewport meta with no zoom.
- Respect `prefers-reduced-motion` and the in-game reduced-motion toggle when adding screen shake, flashes, or particles.
- Keep changes minimal and local. Match the existing naming style (camelCase functions and variables, `UPPER_SNAKE` for constants).

## Git

- Development branch: `claude/gifted-tesla-886fgr`. Commit and push there. Do not push to other branches without explicit permission.
- Do not open a pull request unless the user asks for one.
