# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Google_4 is a small, dependency-free browser game: **Sky Fighter**, a Raiden-style
vertical shoot-'em-up. The entire game lives in one file, `index.html`
(about 1,670 lines: inline CSS, inline JavaScript, no build step, no external assets).
`README.md` is a one-line stub.

Files:

- `index.html`: the whole game (markup, `<style>`, and a single `<script>`).
- `README.md`: project title only.
- `CLAUDE.md`: this file.

## Running the game

No build, install, or test tooling exists. Open `index.html` directly in a browser,
or serve the directory with any static server (for example `python3 -m http.server`).

There is no test suite, linter, or CI configuration in the repo. Verify changes by
playing them in a browser, and check the browser console for errors.

## Code map (`index.html`)

Top-level sections of the `<script>`, in file order:

- **Persistence keys**: `STORAGE_*` constants (`skyfighter.bestScore`, `skyfighter.muted`,
  `skyfighter.difficulty`, `skyfighter.reducedMotion`). Values are read from `localStorage`
  at startup.
- **Adaptive performance**: `trackPerf(dt)`.
- **Tuning tables**: `DIFFICULTIES` (easy/normal/hard: spawn rates, enemy speed, shot chance,
  lives, bombs, boss HP multiplier), `KILLS_PER_STAGE`, `THEMES` (per-stage sky palettes),
  `ENEMY_TYPES`, `BOSSES`.
- **Audio**: WebAudio synthesis only (`ensureAudio`, `beep`, `noiseBurst`). No audio files.
- **HUD and feedback**: `renderHearts`, `renderBombs`, flash/shake/combo/stage-banner helpers.
- **Canvas sizing**: `BASE_W = 480`, `BASE_H = 720`, scaled in `resize()`.
- **Input**: keyboard (`keys[e.code]`), touch joystick, and pointer handling.
  Pause is `P`/`Escape`; bomb is `X`/`Shift`.
- **Game state and logic**: `resetGame`, `spawnEnemy`, `spawnBoss`, `update(dt)`,
  `fireWeapon`, `useBomb`, `applyPowerUp`, `loseLife`, `endGame`, `togglePause`.
- **Boss attacks**: `BOSS_PATTERNS` and the `fire*` pattern helpers, driven by `updateBoss`.
- **Rendering**: `drawPlane`, `drawBossShip`, `drawClouds`, `draw()`.
- **Main loop**: `loop(now)` at the end of the script.

## Conventions

- Keep the game as a single self-contained HTML file unless the user asks for a split.
  Do not add npm dependencies, bundlers, or external CDN scripts without asking.
- Put new tuning values in the existing tables (`DIFFICULTIES`, `THEMES`, `ENEMY_TYPES`,
  `BOSSES`) rather than hard-coding them in logic.
- Persisted settings must use the `skyfighter.*` key prefix and be read/written with
  `localStorage` as the existing code does.
- Respect the reduced-motion setting when adding screen shake, flashes, or other
  motion effects (see `reducedMotion` and `updateMotionBtn`).
- Keep the layout working at phone widths: the page uses `touch-action: none`, safe-area
  insets, and a fixed-aspect canvas. Do not introduce page scrolling.
- Match the existing style: 2-space indentation, single quotes in JS, `const`/`let`, and
  section comments in the `// ---------- Name ----------` form.

## Git

- Develop on the branch designated for this work (currently `claude/gifted-tesla-1okafj`).
- Do not open pull requests unless explicitly asked.
