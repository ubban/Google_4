# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a single-page browser game, **Sky Fighter**: a Raiden-style vertical shoot-'em-up. Switch weapons, drop smart bombs, and take down stage bosses.

The entire game is one self-contained file, `index.html` (~1,670 lines):

- `<style>` (lines ~10–361): all CSS, including the HUD, overlays, touch controls, and the `body.reduced-motion` overrides.
- `<body>` markup (~363–420): the `<canvas id="gameCanvas">`, HUD elements, overlay card, and touch controls.
- `<script>` (~422–1668): one IIFE containing all game logic, rendering, audio, and input.

`README.md` is a stub. There is no build step, package manager, test suite, or linter.

## Running the game

No build is required. Open `index.html` directly in a browser, or serve the directory with any static server, for example:

```bash
python3 -m http.server 8000   # then visit http://localhost:8000/
```

## Code map (inside the `<script>` IIFE)

| Area | Key symbols |
|---|---|
| Config | `DIFFICULTIES`, `KILLS_PER_STAGE` (18), `THEMES` (per-stage sky palettes), `ENEMY_TYPES`, `BOSSES` |
| Persistence | `STORAGE_BEST`, `STORAGE_MUTE`, `STORAGE_DIFF`, `STORAGE_REDUCED` (all `localStorage` keys prefixed `skyfighter.`) |
| Performance | `trackPerf` adapts a `quality` level (0–2) from a 45-frame rolling average of frame time |
| Audio | `ensureAudio`, `beep`, `noiseBurst`, `sfx` (Web Audio API, synthesized, no audio files) |
| Input | `keys` map, `joystickStart/Move/End`, mouse handlers on the canvas, touch buttons `fireBtn` / `bombBtn` |
| Layout | `BASE_W = 480`, `BASE_H = 720`, `resize()` scales the canvas to the viewport |
| Game state | `resetGame()`, `update(dt)`, `draw()`, `loop(now)` (requestAnimationFrame; `dt` capped at 0.05 s) |
| Combat | `fireWeapon`, `useBomb`, `addScore`, `breakChain`, `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`, `applyPowerUp` |
| Boss attacks | `BOSS_PATTERNS` and the `fire*` helpers (radial, aimed spread, columns, spiral, cross, homing, double ring) driven by `updateBoss` |
| Lifecycle | `loseLife`, `killBoss`, `endGame`, `togglePause` |

The game loop is a single `requestAnimationFrame` callback. Game logic runs in `update(dt)` and drawing in `draw()`, and both run only while `running && !paused`.

## Controls

- **Move:** Arrow keys or WASD; touch joystick on touch devices; mouse movement over the canvas.
- **Fire:** Space (held), or the on-screen fire button.
- **Smart bomb:** `X` or Shift, or the on-screen bomb button.
- **Pause:** `P` or `Escape`.
- **Mute / reduced-motion toggles:** the on-screen buttons in the HUD.

## Conventions

- **Keep everything in `index.html`.** Do not split out modules, add a bundler, or introduce a dependency unless the user asks. The single-file, zero-dependency shape is deliberate.
- **Match the existing style:** 2-space indentation, single quotes in JS, `const`/`let` (no `var`), arrow functions for callbacks, and section comments in the form `// ---------- Name ----------`.
- **Keep the IIFE and `'use strict'`.** Module-level state lives inside the IIFE; do not add globals.
- **Tune balance through the config tables** (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `THEMES`) rather than hard-coding magic numbers in logic.
- **Respect accessibility hooks.** Screen-shake, flashes, and pulses should go through `screenShake`, `flashHit`, `flashBomb`, or the `reduced-motion` body class. Announcements for screen readers go through `announce()` into the `#srLive` live region.
- **Persist settings only via the `skyfighter.*` keys** already defined, and wrap new `localStorage` reads in a way that tolerates unavailable storage if you add any.
- **Mobile-first:** the layout must work at phone width, including touch controls (`touchControls`, `.enabled` class added for touch devices). Verify that `touch-action` and `preventDefault` behavior still holds when changing touch code.

## Verifying changes

There are no automated tests. After a change, check it manually in a browser:

1. Load the page, start a game, and confirm the HUD updates (score, hearts, bombs, weapon).
2. Exercise the changed path: firing, bombs, a boss fight, pause/resume, game over, and restart.
3. Test the difficulty buttons and the mute and reduced-motion toggles if the change touches them.
4. For layout or touch changes, check a narrow viewport and touch emulation.

Dev-tools console errors are the quickest signal of a regression, so keep the console clean.

## Git

- Development branch for this session: `claude/gifted-tesla-as6nqn`.
- Commit messages in this repo are short, imperative summaries (see `git log`).
