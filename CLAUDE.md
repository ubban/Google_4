# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo contains **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs as a single self-contained HTML5 canvas page.

- `index.html` — the entire game: markup, CSS (`<style>` block, ~lines 10–362), canvas element, and a single inline IIFE `<script>` (~lines 422–end) containing all game logic.
- `README.md` — minimal placeholder; no build or setup instructions.

There is no build system, package manager, bundler, test suite, or linter. Nothing needs to be installed.

## Running the game

Open `index.html` directly in a browser, or serve the directory with any static server, for example:

```bash
python3 -m http.server 8000   # then visit http://localhost:8000/
```

## Architecture notes (`index.html`)

- **Rendering:** a single `<canvas id="gameCanvas">` drawn with the 2D context. Logical resolution is `BASE_W = 480` × `BASE_H = 720`; `resize()` scales it to the viewport.
- **Main loop:** `update(dt)` advances state, `draw()` renders, and `loop(now)` drives both via `requestAnimationFrame`.
- **Adaptive quality:** `trackPerf()` samples frame times and steps a `quality` level (0 full, 1 medium, 2 low) to shed effects on slow devices.
- **Tuning data lives in constants near the top of the script:**
  - `DIFFICULTIES` — spawn rates, enemy speed and fire chance, starting lives and bombs, boss HP multiplier (`easy` / `normal` / `hard`).
  - `KILLS_PER_STAGE` — kills needed to advance a stage and trigger a boss.
  - `THEMES` — per-stage sky gradient, cloud and sun colors, and theme name.
  - `ENEMY_TYPES` — `grunt`, `interceptor` (weaving), `tank`; size, HP, speed, score, and the earliest stage each appears.
  - `BOSSES` — boss names, colors, and HP multiplier; attack patterns are in `BOSS_PATTERNS`.
- **Weapons and power-ups:** `fireWeapon()` implements Vulcan and Laser with three power levels; `applyPowerUp()` handles weapon level, bomb, and extra life pickups.
- **Bombs:** `useBomb()` clears the screen and damages bosses. Limited per difficulty.
- **Scoring:** `addScore()` applies a kill-chain multiplier, which `breakChain()` resets.
- **Audio:** WebAudio, created lazily by `ensureAudio()` (browsers require a user gesture first). Helpers are `beep()` and `noiseBurst()`. Mute state is persisted.
- **Input:**
  - Keyboard: arrows/WASD move, Space fires, X or Shift bombs, P or Escape pauses.
  - Mouse/pointer: move to steer, click to fire.
  - Touch: on-screen joystick, fire, and bomb buttons, shown when `isTouchDevice` is true.
- **Persistence:** `localStorage` only, with keys prefixed `skyfighter.`:
  - `skyfighter.bestScore`
  - `skyfighter.muted`
  - `skyfighter.difficulty`
  - `skyfighter.reducedMotion`
- **Accessibility:** a `prefers-reduced-motion` setting disables screen shake and similar effects (`body.reduced-motion`). Announcements go through the `srLive` ARIA live region via `announce()`.

## Conventions

- Keep the game a **single self-contained file** unless the user asks for a restructure. Do not add a build step or external dependencies without a clear reason.
- Preserve the existing style: `'use strict'` inside the IIFE, `const`/`let`, two-space indentation, and the section comment banners (`// ---------- Name ----------`).
- Tune gameplay by editing the constant tables above rather than hard-coding numbers in logic.
- Any new persisted setting should use a `skyfighter.`-prefixed key and be read defensively (fall back to a default when absent).
- Touch and keyboard controls must both keep working. Test both paths when changing input handling.
- Respect `reducedMotion` for any new screen shake, flashes, or large animations.

## Verifying changes

There are no automated tests. After editing `index.html`:

1. Load it in a browser and confirm there are no console errors.
2. Play through start, firing, bombs, a power-up, a boss fight, pause/resume, and game over.
3. Check the difficulty and mute toggles, and confirm best score persists across reloads.
4. For layout or input changes, check a narrow mobile viewport with touch controls.

A headless Chromium with Playwright can be used for smoke-testing if needed.

## Git

- Development branch for this work: `claude/gifted-tesla-zcvpok`.
- Keep commits focused, with a short imperative subject line.
