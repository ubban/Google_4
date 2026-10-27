# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Google_4 is a single-page browser game, **Sky Fighter**: a Raiden-style vertical shoot-em-up
rendered on a `<canvas>`. The whole game lives in one file, `index.html`. `README.md` is a stub.

There is no build step, no package manager, no dependencies, no test suite, and no linter.

## Running the game

- Open `index.html` directly in a browser, or serve the directory with any static server
  (for example `python3 -m http.server` and visit `http://localhost:8000`).
- Verify changes by playing them in a browser. Nothing else in the repo checks behavior.

## File layout (`index.html`)

- Lines ~10–360: inline `<style>` (layout, HUD, overlays, touch controls, `reduced-motion` rules).
- Lines ~361–421: `<body>` markup (canvas, HUD, start/pause overlay, touch controls, live region).
- Lines ~422–1667: one inline `<script>`, wrapped in an IIFE with `'use strict'`. Nothing is
  exported to `window`.

Line numbers drift as the file changes. Search for the section comments instead
(`// ---------- Input ----------`, `// ---------- Canvas sizing ----------`, and so on).

Sections inside the script, in order:

1. DOM references and `STORAGE_*` keys
2. Adaptive performance (`quality`, `trackPerf`)
3. Tuning tables: `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES`, `ENEMY_TYPES`, `BOSSES`
4. Audio (`ensureAudio`, `beep`, `noiseBurst`, `sfx`)
5. HUD helpers (hearts, bombs, combo pop, stage banner, screen shake)
6. Canvas sizing (`BASE_W = 480`, `BASE_H = 720`, scaled to fit the window up to 1.6×)
7. Input (keyboard, touch joystick and buttons, mouse)
8. Game state (`resetGame`), drawing (`drawPlane`, `drawBossShip`, `draw`)
9. Simulation (`update`, `spawnEnemy`, `spawnBoss`, `applyPowerUp`, `useBomb`, `fireWeapon`)
10. Boss attack patterns (`BOSS_PATTERNS`, `updateBoss`, `killBoss`)
11. Lifecycle (`loseLife`, `endGame`, `togglePause`) and the `loop(now)` / start wiring at the bottom

## Game design facts

- Difficulty (`easy` / `normal` / `hard`) sets spawn rates, enemy speed, enemy fire chance,
  starting lives and bombs, and a boss HP multiplier. The difficulty buttons are ignored during
  an active, unpaused run.
- A stage ends after `KILLS_PER_STAGE` (18) kills, then a boss spawns after a short delay.
  Bosses cycle through `BOSSES`, and the background cycles through `THEMES`.
- Enemy types: `grunt`, `interceptor` (weaving), `tank` (4 HP, unlocked at stage 2).
- Persistence uses `localStorage` under the `skyfighter.*` keys: best score, mute,
  difficulty, and reduced motion. Every access must tolerate storage being unavailable.

## Controls (keep the in-game hint in sync)

- Keyboard: Arrows or WASD move, Space fires, X or Shift drops a bomb, P or Esc pauses.
- Mouse: move to steer, click to fire.
- Touch: drag in the left zone to fly, FIRE and BOMB buttons on the right.

The controls hint is the `#controlsHint` paragraph in the start overlay. Update it whenever
bindings change.

## Conventions

- Match the existing style: 2-space indent, single quotes, `const` by default, small named
  helper functions, and short `// ---------- Section ----------` banners.
- Keep everything in `index.html` unless there is a clear reason to split it. If you do split
  it, update this file.
- Keep gameplay tuning in the constant tables near the top of the script, not inside the
  update loop.
- Accessibility is part of the feature set. Keep the `aria-live` announcements
  (`announce()`), the `reduced-motion` class, and the `prefers-reduced-motion` default working
  when you change HUD, effects, or animations.
- Keep the layout working at phone width and with touch input. Do not add features that only
  work with a mouse or keyboard.
- Respect the frame budget. The adaptive quality logic drops effects when frames run slow, so
  new per-frame work should be cheap or gated by `quality`.
- Do not add external CDN scripts, fonts, or network calls. The game is self-contained.

## Verification checklist

Because there are no automated checks, do this manually after non-trivial changes:

1. Open `index.html` in a desktop browser and check the console for errors.
2. Start a run, fire, collect a power-up, use a bomb, and reach a boss.
3. Lose all lives and confirm the game-over screen shows the correct best score.
4. Pause and resume with P or Esc, and check that the start overlay button reads "Resume".
5. Toggle mute, reduced motion, and difficulty, reload, and confirm they persist.
6. For layout or input changes, test a narrow viewport or touch emulation.

## Git

- Develop on the branch assigned for the session. Do not push to `main` directly.
- Commit messages are short and imperative, for example `Add ...`, `Fix ...`, `Tune ...`.
