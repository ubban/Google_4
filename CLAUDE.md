# CLAUDE.md

Guidance for working in this repository.

## Project overview

`Google_4` is a small static web project. Its only game is **Sky Fighter**, a
Raiden-style vertical shoot-em-up built as a single self-contained HTML5 canvas
page.

## Repository layout

```
index.html   # The entire game: markup, CSS (<style>), and JS (<script>)
README.md    # Minimal project readme
CLAUDE.md    # This file
```

There is no build system, package manager, bundler, test suite, or linter.
Nothing needs to be installed. Open `index.html` in a browser to play.

## Architecture of `index.html`

- **Markup and CSS** (roughly lines 1–420): HUD (score, best, multiplier, weapon,
  lives as hearts, bombs), overlay card for title/pause/game-over, touch controls
  (joystick, fire, bomb), boss health bar, and a hidden `aria-live` region
  (`#srLive`) used by `announce()`.
- **Script** (one IIFE starting around line 422, `'use strict'`): all state lives
  in closures. Nothing is exported and there are no globals.
  - Constants: `DIFFICULTIES`, `THEMES` (one per stage, cycled), `ENEMY_TYPES`,
    `BOSSES`, `BOSS_PATTERNS`, `KILLS_PER_STAGE`.
  - Simulation: `update(dt)` advances the world; `spawnEnemy`, `spawnBoss`,
    `updateBoss`, `fireWeapon`, `useBomb`, `applyPowerUp`, `loseLife`, `endGame`.
  - Rendering: `draw()` plus helpers `drawPlane`, `drawBossShip`, `drawClouds`.
  - Main loop: `loop(now)` runs under `requestAnimationFrame`. It also drives
    adaptive quality via `trackPerf`, which steps the `quality` level (0 to 2)
    based on a rolling average of frame times.
  - Audio: Web Audio via `ensureAudio()`, `beep()`, `noiseBurst()`, and the `sfx`
    table. Audio starts only after a user gesture.
- **Persistence** uses `localStorage` with keys under the `skyfighter.` prefix:
  `bestScore`, `muted`, `difficulty`, `reducedMotion`.
- **Coordinate system:** logical play field is `BASE_W` x `BASE_H` = 480 x 720.
  `resize()` scales the canvas to fit the viewport.

## Conventions

- Keep the game in one file unless the user asks for a split. Do not add a build
  step or dependencies without a request.
- Match the existing style: 2-space indent, single quotes, `const` and `let`
  (no `var`), arrow functions for callbacks, and short section comments with
  `// ---------- Name ----------` headers.
- Keep everything inside the IIFE. Do not leak new globals.
- Gameplay tuning belongs in the constant tables (`DIFFICULTIES`, `ENEMY_TYPES`,
  `BOSSES`, `THEMES`), not hard-coded in update logic.
- Respect the accessibility features already present: the `reduced-motion` body
  class and `prefers-reduced-motion`, the `announce()` live region for important
  events, and keyboard plus touch plus mouse input. New UI should keep these
  working.
- Any new effect that shakes the screen or flashes should go through
  `screenShake()` or the existing flash helpers, so reduced-motion can gate it.

## Verifying changes

Because there is no test harness, check changes by opening `index.html` in a
browser (or serving the directory with any static server). Exercise these paths:

1. Start, play, and pause/resume. Confirm the overlay and HUD update.
2. Lose all lives, then confirm the game-over card and that best score persists
   after reload.
3. Switch difficulty and mute, then reload to confirm they persist.
4. Reach a boss (stage progression) and confirm the boss bar appears.
5. On a narrow viewport or with touch emulation, confirm the touch controls and
   that the page does not scroll horizontally.

Note that `localStorage` access is not wrapped in `try/catch`. If a change makes
storage more central, consider guarding it so the game still runs in private
windows or when storage is blocked.

## Git workflow

- Develop on the branch assigned for the session. Do not push to other branches
  without explicit permission.
- Write commit messages in the imperative mood with a short summary line. Add a
  body when the change needs explaining.
- Do not open a pull request unless asked.
