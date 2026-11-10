# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up that runs in the browser.
The entire game lives in a single file, `index.html` (about 1,670 lines): inline
CSS, markup for the HUD and overlays, and one `<script>` block containing the
game loop, rendering, audio, input, and persistence. There is no build step,
no package manager, no bundler, and no test suite.

Files:

- `index.html` — the whole game.
- `README.md` — placeholder title only.
- `CLAUDE.md` — this file.

## Running the game

Open `index.html` directly in a browser, or serve the directory with any static
server, for example `python3 -m http.server` from the repo root, then visit
`http://localhost:8000`. No install step is required.

## Architecture (inside `index.html`)

The script is one IIFE (`(function () { ... })();`, ~line 423–1666), so all
`const`/`let` bindings and functions are private to the page. Key areas, roughly in file order:

- **Persistence keys** (`STORAGE_*`, ~line 463): `skyfighter.bestScore`,
  `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion`. All
  reads and writes go through `localStorage`.
- **Difficulty table** (`DIFFICULTIES`, ~line 485): `easy`, `normal`, `hard`,
  each defining spawn rate, enemy speed, enemy fire chance, starting lives,
  starting bombs, and a boss HP multiplier.
- **Stage and boss config** (`KILLS_PER_STAGE = 18`, `BOSSES`, ~line 491–516):
  each stage is gated by kill count, and each boss has a name, colour palette,
  and HP multiplier. `THEMES` (~line 493) sets per-stage backgrounds.
- **Audio** (`ensureAudio`, `beep`, `noiseBurst`, ~line 545–680): Web Audio
  synthesis only; no audio files. Audio is created lazily on first user input
  because browsers require a gesture.
- **HUD and feedback** (~line 680–740): hearts, bomb pips, hit flash, combo
  popups, stage banners, screen shake, and `announce()` for screen readers.
- **Layout** (`resize`, `BASE_W = 480`, `BASE_H = 720`, ~line 753): the canvas
  uses a fixed logical resolution and is scaled to fit the viewport.
- **Input**: keyboard (`keydown`/`keyup` on `window`, ~line 769), mouse
  steering and fire button, and a touch joystick plus FIRE and BOMB buttons
  (~line 785–835).
- **Game state** (`resetGame`, ~line 875): all mutable game state is reset
  here. Add new state to this function so restarts stay correct.
- **Entities and update** (`spawnEnemy`, `spawnBoss`, `update(dt)`,
  ~line 1022–1300): per-frame `update(dt)`, with `dt` clamped to 50 ms. Collision
  uses `rectsOverlap`, scoring goes through `addScore` with a kill-chain
  multiplier (`breakChain`), weapons through `fireWeapon`, and power-ups through
  `applyPowerUp`.
- **Boss patterns** (`BOSS_PATTERNS`, `updateBoss`, ~line 1324–1440): each
  bullet pattern is a small function (radial, aimed spread, columns, spiral,
  cross, homing, double ring). Patterns are selected per boss phase.
- **Lifecycle** (`loseLife`, `endGame`, `togglePause`, `killBoss`,
  ~line 1441–1505).
- **Rendering** (`drawPlane`, `drawBossShip`, `drawClouds`, `draw`,
  ~line 913–1640): all drawing is done with canvas 2D primitives. No sprite
  images are loaded.
- **Main loop** (`loop(now)`, ~line 1640): `requestAnimationFrame` driver.

## Controls (must stay consistent with the on-screen hint)

| Action  | Keyboard                    | Mouse / touch                 |
|---------|-----------------------------|-------------------------------|
| Move    | Arrows or WASD              | Mouse steers / drag left joystick |
| Fire    | Space (hold)                | Click / hold FIRE button      |
| Bomb    | X or Shift                  | Tap BOMB button               |
| Pause   | P or Esc                    | —                             |
| Mute    | — (`#muteBtn` in the HUD)   | Click `#muteBtn`              |

The controls hint text is in `index.html` near the `#controlsHint` element.
If you change a binding, update that text too.

## Conventions

- **Single file.** Keep the game in `index.html` unless the user asks for a
  split. Do not introduce a build tool or framework without a clear request.
- **No external dependencies.** The page loads no CDN scripts or fonts. Keep
  it that way unless asked.
- **Vanilla JS and canvas.** Match the existing style: `const` for tuning
  tables, small named functions, and no classes.
- **Respect `reducedMotion`.** Screen shake, flashes, and similar effects
  should check the reduced-motion flag before running.
- **Keep mobile working.** Touch handlers use `{ passive: false }` where they
  call `preventDefault`. Test layout at phone width when changing HUD or
  overlay CSS.
- **Accessibility.** Use `announce()` for important state changes, and keep
  overlay text in real DOM elements rather than only on canvas.
- **Persistence is local only.** Do not add network calls for scores.

## Making changes

1. Read the relevant section before editing; line numbers above are approximate
   and shift as the file changes.
2. Keep diffs small and local. Prefer editing an existing function over
   adding parallel code paths.
3. Verify by playing: load the page, start a run, and exercise the changed
   behaviour with keyboard and, if relevant, touch emulation. There are no
   automated tests or linters in this repo.
4. Check the browser console for errors after any change to the script block.

## Git

- Develop on the branch assigned to the session (currently
  `claude/gifted-tesla-se5bt2`). Do not push to other branches without
  explicit permission.
- Write descriptive commit messages in the imperative mood.
