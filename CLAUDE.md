# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo is a small, dependency-free browser game, **Sky Fighter**: a Raiden-style vertical shoot-'em-up. The entire game lives in one file, `index.html` (~1,670 lines: inline `<style>`, markup, and one inline `<script>` IIFE). There is no build step, no package manager, no test suite, and no external assets or CDN dependencies.

- `index.html` — the whole game (HTML5 canvas, 2D context, Web Audio for sound effects).
- `README.md` — minimal placeholder; currently just the repo title.

The game's logical canvas is `BASE_W = 480` × `BASE_H = 720` and is scaled to fit the viewport.

## Running and verifying

- Open `index.html` directly in a browser, or serve the directory with any static server (e.g. `python3 -m http.server`).
- There are no automated tests or linters configured. Verify changes by playing the game in a browser, ideally at both desktop and phone-sized viewports.
- If you have Playwright available, the pre-installed Chromium can be used for a headless smoke test; do not run `playwright install`.

## Code layout inside `index.html`

The script is wrapped in a single `(function () { 'use strict'; ... })();` IIFE. Within it, roughly in order:

1. **DOM lookups** — every element is fetched once by `id` at the top (`gameCanvas`, `overlay`, `hud` pieces, touch controls, buttons, etc.). If you add a UI element, add its lookup here.
2. **Persistence keys** — `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion`. All `localStorage` access is direct (no try/catch wrapper); keep new keys namespaced under `skyfighter.`.
3. **Adaptive performance** — `trackPerf` adjusts a `quality` level (0–2) from rolling frame times.
4. **Tuning tables** — these are the first place to look when balancing:
   - `DIFFICULTIES` (`easy` / `normal` / `hard`): spawn rates, enemy speed, shoot chance, starting lives and bombs, boss HP multiplier.
   - `KILLS_PER_STAGE` (18): kills needed to advance a stage.
   - `THEMES`: per-stage sky gradient, cloud/sun tints, and names (cycled by stage).
   - `ENEMY_TYPES`: `grunt`, `interceptor`, `tank` (with `minStage` gating).
   - `BOSSES`: six bosses (VANGUARD, BEHEMOTH, SPECTER, DREADNOUGHT, WRAITH, TITAN), each with colors and an `hpMul`.
   - `BOSS_PATTERNS`: per-boss arrays of attack functions (`fireRadial`, `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`, `fireCross`, `fireHomingSeeded`, `fireDoubleRing`, …).
5. **Audio** — `ensureAudio()` lazily creates the `AudioContext` (must be triggered by a user gesture). `beep()` and `noiseBurst()` are the primitives; `sfx.*` wraps them. Sound is muted via the `muted` flag and the mute button.
6. **Input** — keyboard (`keys` map, keydown/keyup on `window`), mouse (`canvasPointFromEvent`, `mouseActive`), and touch (virtual joystick on `moveZone`, `fireBtn`, `bombBtn`).
7. **Game state** — `resetGame()` initializes the module-level `let` state (`score`, `lives`, `running`, `paused`, `gameOver`, etc.). Entities are plain objects in arrays.
8. **Update / draw** — `update(dt)` is the per-frame simulation (around line 1150); drawing helpers (`drawPlane`, `drawBossShip`, …) follow. The main loop calls update then render.
9. **Power-ups and weapons** — `applyPowerUp`, `fireWeapon`, weapon levels (Vulcan / Laser, 3 levels each), `useBomb()` for smart bombs.

## Controls (keep the in-game hint in sync)

The hint text lives in `#controlsHint` in the markup. Current bindings:

- Mouse: move to steer, click to fire.
- Keyboard: Arrows or WASD to move, Space to fire, X or Shift to bomb, P or Esc to pause.
- Touch: drag left to fly, tap FIRE, tap BOMB.

If you change a binding, update both the keydown handler (around line 769) and `#controlsHint`.

## Conventions

- Keep the game a single self-contained HTML file unless the user asks to split it. Don't introduce a bundler, framework, or npm dependency without an explicit request.
- Stick to the existing style: `const` by default, `let` for mutable game state, arrow functions for callbacks, two-space indentation, and the existing section comment banners (`// ---------- Name ----------`).
- Respect `reducedMotion` for any new screen-shake, flash, or particle effects (see `screenShake` and `flashHit`).
- Preserve accessibility hooks: keep the `srLive` announcements for stage and game-state changes, and keep button `title`/labels current.
- Preserve mobile behavior: `touch-action: none`, safe-area insets, and the `resize()` logic that scales to `BASE_W × BASE_H`.
- Do not add external network requests, analytics, or third-party scripts.

## Git

- Development branch for this repo: `claude/gifted-tesla-78ud0k`. Commit and push there; don't push to other branches without permission.
- Don't open a pull request unless the user asks for one.
