# CLAUDE.md

Guidance for Claude Code when working in this repository.

## What this is

`Sky Fighter` — a Raiden-style vertical shoot-'em-up that runs in the browser on an HTML5 `<canvas>`. The whole game is one file: `index.html` (~1,670 lines: inline `<style>`, markup, and one inline `<script>` IIFE). There is no build step, no package manager, no test suite, and no external dependencies.

`README.md` is a placeholder and does not describe the game.

## Running it

- Open `index.html` directly in a browser, or serve the folder with any static server (e.g. `python3 -m http.server` and visit `/index.html`).
- Nothing to install or compile.

## Repository layout

```
index.html   # entire game: CSS, DOM/HUD, and JS (all in one file)
README.md    # placeholder
CLAUDE.md    # this file
```

Inside `index.html`:

| Region | Purpose |
|---|---|
| `<style>` (lines ~10–360) | Layout, HUD, overlay, touch controls, `.reduced-motion` and mute states |
| `<body>` (lines ~361–421) | `#container` with canvas, vignette/flash overlays, HUD, boss bar, touch controls, start/pause overlay, difficulty buttons |
| `<script>` (lines ~422–1667) | One `(function () { 'use strict'; ... })();` IIFE holding all game code |

Within the script, the code is grouped by `// ---------- ... ----------` section comments: adaptive performance, audio, canvas sizing, input, game state, drawing, update, bosses, and the main loop.

## Architecture notes

- **Single rAF loop.** `loop(now)` computes `dt` (capped at 0.05 s), calls `trackPerf`, then `update(dt)` and `draw()` only while `running && !paused`.
- **Fixed logical resolution.** `BASE_W = 480`, `BASE_H = 720`. `resize()` scales the canvas CSS size to fit the window (max 1.6×). All game coordinates are in canvas pixels.
- **Adaptive quality.** `trackPerf` keeps a 45-frame rolling average and steps `quality` (0 full / 1 medium / 2 low) down or up based on frame time.
- **Game state is module-level `let`s** inside the IIFE: `player`, `bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`, `boss`, `score`, `lives`, `bombCount`, `stage`, `stageKills`, `chain`, `multiplier`, etc. `resetGame()` is the single place that initializes them; keep it in sync when adding state.
- **Audio** is synthesized with the Web Audio API (`beep`, `noiseBurst`, and an `sfx` object). `ensureAudio()` must run from a user gesture before sound plays.
- **Persistence** uses `localStorage` with `skyfighter.*` keys (`bestScore`, `muted`, `difficulty`, `reducedMotion`). Every access is currently unguarded; wrap new ones in try/catch if they might run in restricted contexts.
- **Accessibility.** `announce(text)` writes to the `#srLive` aria-live region. Use it for important state changes (lives, game over, stage). `prefers-reduced-motion` seeds the reduced-motion toggle and disables screen shake and flashes.

## Tuning data (edit these, not the logic)

All balance lives in constant tables near the top of the script (roughly lines 485–527):

- `DIFFICULTIES` — `easy` / `normal` / `hard`: spawn timing, enemy speed and shoot chance, starting `lives`, `bombs`, and `bossHpMul`.
- `KILLS_PER_STAGE = 18` — kills needed before a boss is spawned (boss spawns 1.2 s after `bossWarned`).
- `THEMES` — six stage backgrounds, cycled per stage. Each has `sky`, `cloud`, `sun` colors and a `name`.
- `ENEMY_TYPES` — `grunt`, `interceptor` (weaves), `tank` (4 HP, unlocks at stage 2). Each has size, HP, speed multiplier, colors, score, and `minStage`.
- `BOSSES` — six bosses (`VANGUARD`, `BEHEMOTH`, `SPECTER`, `DREADNOUGHT`, `WRAITH`, `TITAN`), chosen by stage. Each has colors and an `hpMul`.
- `BOSS_PATTERNS` (~line 1372) — the attack patterns bosses cycle through (radial, aimed spread, columns, spiral, cross, homing, double ring).

## Controls (keep the in-game hint in sync)

The hint text lives in `#controlsHint` (line ~417). Current bindings:

- Mouse: move to steer, click to fire. Arrows/WASD move. Space fires.
- X or Shift: bomb. P or Esc: pause.
- Touch: drag left side to fly (virtual joystick), tap FIRE, tap BOMB.

Power-ups dropped by enemies (`maybeSpawnPowerUp`, `applyPowerUp`): `life` (red), `bomb` (amber), `weapon` (green, upgrades the weapon level).

## Conventions

- Keep everything in `index.html` unless there is a clear reason to split it. The no-build, open-and-play property is a deliberate feature.
- Match the existing style: 2-space indentation, `const`/`let`, arrow callbacks, `'use strict'` inside the IIFE, and the `// ---------- Section ----------` comment banners.
- Don't add dependencies, CDN scripts, or a bundler without asking.
- Preserve the `localStorage` key names; changing them silently resets players' best scores and settings.
- When adding a control, update the `#controlsHint` text, the touch controls if relevant, and the `keydown` handler together.
- When adding a new enemy, boss, or power-up type, add it to its table, make sure `update`/`draw` handle it, and check `resetGame()` if it carries state.

## Verifying changes

There are no automated tests or linters. To verify a change:

1. Open `index.html` in a browser (or serve it) and check the browser console for errors.
2. Play through at least one full stage to a boss on each difficulty you touched.
3. Check the mobile layout and touch controls at a narrow viewport if you changed HUD or input code.
4. Toggle mute, reduced motion, and pause, and confirm best score persists across reloads.

If you add a test harness, document how to run it here.

## Git

- Default branch is `main`. Feature work for this session is on `claude/gifted-tesla-qtuq9n`.
- Keep commits focused, with imperative subject lines (the existing history uses e.g. "Add Sky Fighter: ...").
