# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Google_4 is a small static web project. Its only real content is **Sky Fighter**, a
Raiden-style vertical shoot-em-up built as a single-file HTML5 canvas app.

- `index.html` — the whole game: markup, CSS (in a `<style>` block), and JavaScript
  (in one `<script>` block wrapped in an IIFE with `'use strict'`).
- `README.md` — placeholder; no real project documentation yet.

There is no build step, no package manager, no bundler, and no test suite.

## Running the game

Open `index.html` directly in a browser, or serve the directory with any static server
(for example `python3 -m http.server`). Nothing needs to be installed.

## Code layout inside `index.html`

The script is organized by section comments (e.g. `// ---------- Adaptive performance ----------`).
Keep new code in the matching section, and add a new section comment when adding a new area.

Key areas, roughly in file order:

- **DOM references and storage keys** — `getElementById` lookups at the top; `STORAGE_*`
  constants prefixed `skyfighter.` for `localStorage`.
- **Adaptive performance** — `trackPerf` adjusts a `quality` level (0–2) from frame times.
- **Config tables** — `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES` (per-stage sky palettes),
  `ENEMY_TYPES`, `BOSSES`. Tune gameplay here rather than hard-coding numbers in logic.
- **Audio** — lazily created `AudioContext` via `ensureAudio()`; sound effects are built
  from `beep()` and `noiseBurst()`. Respect the `muted` flag.
- **HUD and feedback** — `renderHearts`, `renderBombs`, `showComboPop`, `showStageBanner`,
  `screenShake`, `flashHit`/`flashBomb`, `announce` (screen-reader live region).
- **Input** — keyboard (`keys` map), touch joystick and buttons, and mouse on the canvas.
- **Game state and logic** — `resetGame`, `update(dt)`, spawning, power-ups, bombs,
  weapons (`fireWeapon`), score multiplier (`addScore`, `breakChain`), boss patterns
  (`BOSS_PATTERNS`, `updateBoss`), `loseLife`, `endGame`, `togglePause`.
- **Rendering** — `draw()` and the helpers `drawPlane`, `drawBossShip`, `drawClouds`.
- **Main loop** — `loop(now)` drives `update` and `draw` via `requestAnimationFrame`.

## Conventions

- Plain JavaScript (ES2015+), no transpilation, no libraries. Do not add external
  dependencies or CDN scripts without a clear reason.
- Use `const` by default; `let` only when reassigned. Keep the existing 2-space indentation
  and single quotes for strings.
- Match the surrounding style for naming: `UPPER_SNAKE_CASE` for constant tables,
  `camelCase` for functions and variables, `STORAGE_*` for persistence keys.
- The game uses a fixed logical resolution (`BASE_W = 480`, `BASE_H = 720`) scaled by
  `resize()`. Draw and position in logical coordinates.
- Persisted settings (best score, mute, difficulty, reduced motion) go through
  `localStorage` with the `skyfighter.` prefix. Wrap new storage access so the game still
  works when storage is unavailable (private mode, blocked site data).
- Support both keyboard and touch input when adding controls. The HUD and overlay must
  stay usable at phone width.
- Preserve accessibility hooks: honor `prefers-reduced-motion` (the `reducedMotion` flag
  and `reduced-motion` body class) for screen shake and flashes, and announce important
  state changes through `announce()`.

## Working in this repo

- Make changes in `index.html` unless a task clearly calls for a new file. If a new file is
  added, reference it with a relative path and no leading slash.
- There are no automated tests or linters. Verify changes by loading the page in a browser
  and playing through the affected behavior (start, fire, bomb, pause, game over, and the
  mobile touch layout when UI changes).
- Keep commits focused. Write messages that describe the gameplay or code change, not the
  file touched.
- Do not commit generated files, editor state, or `node_modules`.

## Git

- Development branch for this session: `claude/gifted-tesla-lutkba`.
- Push with `git push -u origin <branch>`. Do not open a pull request unless explicitly asked.
