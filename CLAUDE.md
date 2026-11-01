# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Google_4 is a small static web project. Its only real content is **Sky Fighter**, a
Raiden-style vertical shoot-em-up built as a single-page HTML5 canvas game.

| File        | Purpose                                                              |
|-------------|----------------------------------------------------------------------|
| `index.html`| The entire game: markup, CSS (`<style>`), and JavaScript (`<script>`) in one file (~1,670 lines). |
| `README.md` | Minimal project readme (title only).                                 |

There is no build step, package manager, bundler, test suite, or linter configured.
Do not add one unless the user asks.

## Running the game

Open `index.html` directly in a browser, or serve the directory with any static
server (for example `python3 -m http.server` from the repo root) and visit
`http://localhost:8000`.

## Code structure of `index.html`

- **`<style>`** (roughly lines 10–361): HUD pills, glassmorphic overlay card, touch
  controls, and reduced-motion styles. The `body.reduced-motion` class disables
  screen shake, flash, and pulse effects.
- **`<body>` markup** (roughly lines 362–421): `#container` holds the canvas
  (`#gameCanvas`), HUD (`#hud`, `#topBar`), boss bar, combo/stage banners,
  touch controls, and the start/pause overlay (`#overlay`).
- **`<script>`** (starts around line 422): one IIFE (`'use strict'`) containing
  all game logic. Key sections, in order:
  1. DOM references (`const xxxEl = document.getElementById(...)`).
  2. Persistence keys (`STORAGE_*`) and tuning tables: `DIFFICULTIES`,
     `THEMES` (per-stage palettes), `ENEMY_TYPES`, `BOSSES`, `KILLS_PER_STAGE`.
  3. Adaptive performance (`trackPerf`, `quality` 0–2) that lowers effect detail
     when frame time is high.
  4. Audio (`ensureAudio`, `beep`, `noiseBurst`), synthesized with Web Audio. No audio files.
  5. HUD helpers (hearts, bombs, combo pops, stage banners, screen shake).
  6. Input: keyboard, mouse, and touch joystick (`keys`, `firing`, `mouseActive`).
  7. Game state and lifecycle (`resetGame`, `update(dt)`, `loop(now)`, `togglePause`).
  8. Rendering (`drawPlane`, `drawBossShip`, and the rest of the draw helpers).
  9. Spawning and weapons (`spawnEnemy`, `spawnBoss`, `fireWeapon`, `useBomb`,
     `applyPowerUp`).
  10. Boss attack patterns (`BOSS_PATTERNS`, `fireRadial`, `fireCross`, and others).

Run the loop with `requestAnimationFrame`. `update(dt)` runs the simulation and
the draw functions render each frame. Keep the two separate.

## Persistence

Player settings and the high score are stored in `localStorage` under keys
prefixed with `skyfighter.`:

- `skyfighter.bestScore`
- `skyfighter.muted`
- `skyfighter.difficulty`
- `skyfighter.reducedMotion`

Changing a key name or value format will reset existing players' saved data.
Treat those keys as a public contract and avoid renaming them without a migration.

## Conventions

- **Single file.** Keep HTML, CSS, and JS in `index.html` unless the user asks to
  split them. There is no module system.
- **Vanilla JS only.** Do not add frameworks or CDN dependencies without asking.
- **Tuning lives in constants.** Change difficulty, enemy, boss, and theme numbers in
  the tables near the top of the script, not inline in logic.
- **Frame-rate independence.** Movement and timers use the `dt` passed to `update`.
  Follow that pattern for new entities.
- **Accessibility.** New visual effects should respect `reducedMotion`. Announce
  important state changes through `announce()`, which feeds the `#srLive` region.
- **Touch and desktop.** Every new control needs both a keyboard/mouse path and a
  touch path. The overlay's controls hint text (`#controlsHint`) should match the
  actual bindings.
- **Keep the BASE canvas size.** The game logical size is `BASE_W` × `BASE_H`
  (480 × 720). `resize()` scales it to fit the window.

## Controls (for reference when changing input)

- Mouse: move to steer, click to fire.
- Keyboard: Arrows or WASD to move, Space to fire, X or Shift for bomb, P or Esc to pause.
- Touch: drag left zone to fly, FIRE and BOMB buttons on the right.

## Git workflow

- Develop on the branch the session was given (currently `claude/gifted-tesla-i7qkh8`).
- Commit with clear messages that describe the gameplay or code change.
- Do not open a pull request unless the user asks for one.
- Do not commit generated or editor-specific files.

## Verifying changes

There are no automated tests. After changing game logic, rendering, or input:

1. Open `index.html` in a browser (or a local static server) and check the browser
   console for errors.
2. Play through at least one stage on Normal, and check the boss fight if the change
   touches bosses, weapons, or power-ups.
3. Check the touch layout at phone width if the change touches HUD or controls.
4. Confirm pause (P/Esc), mute, reduced-motion, and difficulty buttons still work if
   the change touches those systems.

Report what was and was not verified. Visual or gameplay changes have not been
checked until they were run in a browser.
