# CLAUDE.md

Guidance for Claude Code when working in this repository.

## What this repo is

`Google_4` is a small static web project containing one game: **Sky Fighter**, a Raiden-style
vertical shoot-'em-up rendered on a `<canvas>`.

- `index.html` — the whole game: markup, CSS (`<style>`), and JavaScript (`<script>`) in one file (~1,670 lines).
- `README.md` — placeholder title only.

There is no package manager, build step, bundler, test suite, linter, or CI config. Do not add
one unless asked.

## Running it

Open `index.html` directly in a browser, or serve the directory with any static server, e.g.:

```bash
python3 -m http.server 8000   # then visit http://localhost:8000/
```

There are no automated tests. Verify changes by playing the game in a browser (keyboard, mouse,
and touch controls all exist; see below).

## Code layout inside `index.html`

- **Markup**: `#gameCanvas`, `#topBar` (mute / reduced-motion / pause buttons), `#touchControls`
  (joystick zone, FIRE, BOMB), `#overlay` (title, difficulty buttons, Start button, controls hint),
  plus HUD and effect elements (`#hearts`, `#bombs`, `#comboPop`, `#stageBanner`, `#flash`, `#vignette`, boss bar).
- **Script**: a single IIFE with `'use strict'`. Everything lives inside it; nothing is exported or
  attached to `window`. Keep new code inside the IIFE.
- **Constants** (top of script): `DIFFICULTIES` (easy/normal/hard tuning), `THEMES` (per-stage sky
  palettes), `ENEMY_TYPES` (grunt, interceptor, tank), `BOSSES` (per-stage boss designs), `KILLS_PER_STAGE`,
  `BASE_W` / `BASE_H` (480×720 logical resolution).
- **Persistence**: `localStorage` keys prefixed `skyfighter.` — `bestScore`, `muted`, `difficulty`,
  `reducedMotion`. Reads and writes are not wrapped in try/catch; keep that in mind if you touch them.
- **Audio**: Web Audio API, created lazily by `ensureAudio()` (browsers require a user gesture).
  Sound helpers are `beep`, `noiseBurst`, and the `sfx` table.
- **Rendering**: `resize()` scales the canvas to fit the window (max 1.6×). `draw()` renders each
  frame; `drawPlane` and `drawBossShip` draw the ships. `loop(now)` drives `update(dt)` and `draw()` via
  `requestAnimationFrame`. `trackPerf` adjusts a `quality` level (0 full, 1 medium, 2 low) when frames run slow.
- **Input**: keyboard (`keys` map; arrows/WASD move, Space fire, X/Shift bomb, P/Esc pause), mouse
  (canvas `mousemove` steers the plane, `mousedown` fires), and touch (joystick on the left, FIRE and BOMB buttons).
- **Game state**: module-level `let`s around `resetGame()` — `player`, `bullets`, `enemies`,
  `enemyBullets`, `particles`, `powerUps`, `boss`, `score`, `lives`, `bombCount`, `stage`, `chain`/`multiplier`, etc.
- **Power-ups** (`maybeSpawnPowerUp`, applied around line 1305): `life`, `bomb`, and `weapon`
  (toggles between `vulcan` and `laser`).

## Conventions

- Keep the project a single self-contained HTML file unless the user asks for a split. Don't add
  dependencies or CDN links without a reason.
- Preserve the existing style: 2-space indentation, single quotes in JS, `const` by default, `let` for mutable state.
- Gameplay numbers belong in the constant tables (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `THEMES`)
  rather than scattered as magic numbers.
- Respect the accessibility features already present: the reduced-motion toggle gates screen shake,
  flashes, and pulses; `announce()` writes to the `#srLive` live region. New effects that flash or shake
  should honor the same toggle.
- Mobile matters: keep touch targets usable and the layout working at phone widths (the page disables user scaling).

## Git

- Develop on the branch the task specifies (currently `claude/gifted-tesla-tcklxi`). Don't push to `main` directly.
- Commit messages: short imperative summary line, optional body explaining why.
- Only open a pull request when the user explicitly asks for one.
