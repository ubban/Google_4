# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Sky Fighter, a Raiden-style vertical shoot-'em-up that runs in the browser on a
`<canvas>`. The whole game is one static file, `index.html`. `README.md` is a
placeholder.

There is no build step, package manager, test suite, or linter. Do not add
tooling unless asked.

## File layout (`index.html`, ~1670 lines)

- **Lines ~10–360**: `<style>`. Layout, HUD, overlays, touch controls, and
  `body.reduced-motion` overrides.
- **Lines ~361–421**: markup. `#container` holds the canvas, the HUD
  (`#hud`, `#topBar`, `#bossBarWrap`), the touch controls (`#touchControls`),
  and the `#overlay` title/game-over card.
- **Lines ~422–1668**: one `<script>` containing a single IIFE (`'use strict'`).
  All game state is private to it.

Script sections are marked with `// ---------- Name ----------` comment headers.
Keep new code in the matching section.

## Running and checking changes

- Open `index.html` directly in a browser. No server is needed.
- Verify gameplay changes in a real browser, not by reading code alone. The
  Playwright build pre-installed in the cloud environment works for this.
- Check the browser console for errors after each change.

## Architecture notes

- **Fixed logical resolution.** `BASE_W = 480`, `BASE_H = 720`. `resize()`
  scales the canvas to fit the window. Game coordinates always use the base
  size.
- **Game loop.** `loop(now)` runs on `requestAnimationFrame`. It clamps `dt` to
  0.05 s, calls `trackPerf`, then `update(dt)` and `draw()` while the game is
  running and not paused.
- **Adaptive quality.** `quality` (0 full, 1 medium, 2 low) is stepped up or
  down from the average frame time over the last 45 frames. Use it to reduce
  effects cost rather than adding fixed toggles.
- **Tuning tables.** Balance lives in `const` tables at the top of the script:
  - `DIFFICULTIES`: spawn timing, enemy speed, shoot chance, lives, bombs, and
    boss HP multiplier per `easy` / `normal` / `hard`.
  - `KILLS_PER_STAGE` (18): kills needed to advance a stage.
  - `THEMES`: six sky palettes, cycled by stage.
  - `ENEMY_TYPES`: `grunt`, `interceptor`, and `tank`, with `minStage` gating.
  - `BOSSES`: six bosses, one per stage, cycling with `(stage - 1) % length`.
    Attack patterns live in `BOSS_PATTERNS`, and boss behavior is in
    `updateBoss`.
  Change balance by editing these tables first.
- **Audio.** Web Audio. `ensureAudio()` creates the context on the first user
  gesture, because browsers require one. Keep audio calls behind that guard.
- **Persistence.** `localStorage` keys, all prefixed `skyfighter.`:
  `bestScore`, `muted`, `difficulty`, `reducedMotion`. Each key has a
  `STORAGE_*` constant. Add new keys the same way.
- **Accessibility.** Reduced motion is on if the OS asks for it or the player
  toggles `#motionBtn`. It disables shake, flashes, and pulses through the
  `reduced-motion` body class. New visual effects that shake, flash, or pulse
  must respect it. Screen-reader announcements go through `announce()`, which
  writes to `#srLive`.

## Controls (keep in sync with `#controlsHint`)

- **Keyboard:** arrows/WASD move, Space fires, X or Shift bombs, P or Esc
  pauses.
- **Mouse:** move to steer, hold to fire.
- **Touch:** drag the left zone (joystick), tap FIRE (hold to keep firing),
  tap BOMB.

If you change a binding, update the `keydown` / `keyup` handlers and the
`#controlsHint` text together.

## Code style

- 2-space indentation, single quotes in JS, semicolons, `const` by default.
- camelCase for functions and variables. UPPER_SNAKE_CASE for module-level
  config constants.
- Match the surrounding code's comment density. Comments explain intent or
  game-design reasoning, not what the next line does.
- Keep it a single file. Do not split out modules or add dependencies unless
  asked.

## Git

- Develop on the branch named in the session instructions. Do not push to
  `main` directly.
