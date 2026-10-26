# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Google_4 is a small static web project. Its only real content is **Sky Fighter**, a
Raiden-style vertical shoot-'em-up that runs in the browser on a single HTML canvas.

- `index.html` — the entire game: markup, CSS, and JavaScript in one file (~1,670 lines).
- `README.md` — minimal placeholder.

There is no build step, no package manager, no dependencies, and no test suite.

## Running the game

Open `index.html` directly in a browser, or serve the directory with any static server
(for example `python3 -m http.server` from the repo root) and visit it on localhost.

## Code layout inside `index.html`

- **`<style>`** (around lines 10–360): all styling, including the HUD, overlays,
  touch controls, and a `body.reduced-motion` class.
- **`<body>` markup** (around lines 363–420): the `<canvas id="gameCanvas">`, HUD
  elements, start/pause/game-over overlay, touch controls, and a `#srLive` region for
  screen-reader announcements.
- **`<script>`** (starts around line 422): one IIFE wrapping everything. Keep new code
  inside it so nothing leaks onto `window`.

Within the script, code is grouped under banner comments such as
`// ---------- Adaptive performance ----------`. Keep to that pattern and add new code to
the matching section, or add a new banner for a new area.

Key sections, in order:

| Area | What it holds |
| --- | --- |
| Config constants | `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES`, `ENEMY_TYPES`, `BOSSES`, `BOSS_PATTERNS` |
| Adaptive performance | `trackPerf` lowers `quality` (0–2) when frame time is poor |
| Audio | `ensureAudio`, `beep`, `noiseBurst`, and the `sfx` helpers (Web Audio, no asset files) |
| Input | Keyboard (`keys`), touch joystick and buttons, mouse steering |
| Game state | `resetGame`, `update(dt)`, `loseLife`, `endGame`, `togglePause` |
| Spawning and combat | `spawnEnemy`, `spawnBoss`, `fireWeapon`, `applyPowerUp`, `useBomb`, scoring and combo chain |
| Boss patterns | `fire*` bullet-pattern functions, `updateBoss`, `killBoss` |
| Rendering | `drawPlane`, `drawBossShip`, `drawClouds`, `draw()` |
| Loop | `loop(now)` drives `update` and `draw` through `requestAnimationFrame`; `dt` is clamped to 0.05s |

## Game conventions

- **Logical resolution** is `BASE_W × BASE_H` = 480 × 720. `resize()` scales the canvas
  CSS size to fit the window and caps the scale at 1.6×. Write game coordinates in the
  480 × 720 space.
- **Tuning values** live in the config constants at the top of the script. Prefer
  adjusting those over hard-coding numbers in update or draw logic.
- **Difficulty** (`easy` / `normal` / `hard`) sets spawn rates, enemy speed and fire
  chance, starting lives and bombs, and boss HP multipliers.
- **Persistence** uses `localStorage` with these keys: `skyfighter.bestScore`,
  `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion`. Read and write
  them only through the existing constants. The game must still work if storage is
  unavailable.
- **Accessibility:** respect `prefers-reduced-motion`. The `reducedMotion` flag scales down
  `screenShake` and skips the hit/bomb flash effects. Announce important state changes
  through `announce()`, which feeds the `#srLive` region.
- **Controls** are documented in the `#controlsHint` text. Keep it in sync if you change
  bindings. Current bindings: arrows/WASD to move, Space or mouse click to fire,
  X or Shift for a bomb, P or Esc to pause.
- **Mobile:** touch controls appear only on touch-capable devices (`touchControls.enabled`).
  Test any input change on both mouse and touch paths.
- **Style:** vanilla JavaScript (ES2015+), no frameworks or libraries. Use 2-space
  indentation, single quotes, `const`/`let`, and arrow functions for callbacks. Match the
  surrounding comment density: brief section banners and short notes on non-obvious logic.

## Making changes

- Keep changes small and local. The whole game lives in one file, so check for existing
  helpers (`spawnExplosion`, `screenShake`, `showComboPop`, `showStageBanner`, `sfx.*`)
  before adding new ones.
- There are no automated tests or linters. Verify changes by loading the game in a browser
  and playing through the affected path: start, fire, bomb, pause/resume, a stage change,
  boss fight, and game over. Check the browser console for errors.
- When changing the visuals or HUD, also check a narrow (phone-width) viewport.

## Git

- Develop on the branch named for the task (currently `claude/gifted-tesla-5exm3u`).
  Do not push to `main` without explicit permission.
- Write clear commit messages that describe the change and its reason.
- Do not open a pull request unless the user asks for one.
