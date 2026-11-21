# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a small static web project. Its only real content is **Sky Fighter**, a Raiden-style vertical shoot-'em-up rendered on an HTML5 `<canvas>`.

Files:

- `index.html` — the entire game: inline CSS (`<style>`, ~lines 10–360), markup (`<body>`, ~line 361), and one inline `<script>` (~line 422 to end) wrapped in an IIFE.
- `README.md` — a one-line placeholder title.
- `CLAUDE.md` — this file.

There is no build step, no package manager, no bundler, no test suite, and no linter configuration. Do not invent `npm`/`make` commands; none exist.

## Running and verifying

- Open `index.html` directly in a browser, or serve the directory with any static server (e.g. `python3 -m http.server`) and visit it.
- Verification is manual: play the game in a browser and check the behavior you changed. Syntax errors show up in the browser console.
- For automated checks, a headless Chromium with Playwright is typically available; use it only if the task calls for it.

## Architecture of `index.html`

The script is a single IIFE. Major sections, in source order:

1. **DOM references** — `getElementById` lookups for HUD, overlay, touch controls, and buttons.
2. **Storage keys** — `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion` in `localStorage`. All persisted state uses these keys; keep the prefix when adding new ones.
3. **Tuning data** — `DIFFICULTIES` (spawn rate, enemy speed, shoot chance, lives, bombs, boss HP multiplier), `KILLS_PER_STAGE`, `THEMES` (sky palettes per stage), `ENEMY_TYPES` (grunt, interceptor, tank), `BOSSES` (six boss designs cycled by stage).
4. **Audio** — a lazily created Web Audio context (`ensureAudio`) with helpers `beep`, `noiseBurst`, and a named `sfx` table. Audio is created only after a user gesture.
5. **HUD helpers** — hearts/bombs rendering, hit/bomb flashes, combo pop-ups, stage banner, screen shake.
6. **Input** — keyboard (`keys` map on `keydown`/`keyup`), touch joystick and fire/bomb buttons, and mouse/pointer movement via `canvasPointFromEvent`.
7. **Game state** — module-level `let` variables (`player`, `bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`, `boss`, `score`, `lives`, `stage`, `chain`/`multiplier`, etc.), reset by `resetGame()`.
8. **Gameplay** — `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`, `fireWeapon`, `useBomb`, `update(dt)`, `applyPowerUp`, boss attack patterns (`fireRadial`, `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`, `fireCross`, `fireHomingSeeded`, `fireDoubleRing`), `updateBoss`, `killBoss`, `loseLife`, `endGame`, `togglePause`.
9. **Rendering** — `draw()` with helpers `drawPlane`, `drawBossShip`, `drawClouds`. Drawing is plain Canvas 2D; there are no sprite assets.
10. **Main loop** — `loop(now)` calls `trackPerf(dt)`, then `update` and `draw` when running and not paused. `dt` is clamped to 50 ms. `trackPerf` drops a `quality` level (0 full, 1 medium, 2 low) when frames run slow.

Stage flow: a stage ends after `KILLS_PER_STAGE` kills, then a boss spawns; defeating it increments `stage` (around line 1449). Stage also gates which enemy types can appear via `minStage`.

## Conventions

- **Single file.** Keep HTML, CSS, and JS in `index.html` unless the user asks for a split. Do not add a framework or build tooling without being asked.
- **No external dependencies.** The game uses no CDN scripts or fonts; keep it that way unless requested.
- **Fixed logical resolution.** Game coordinates use `BASE_W = 480`, `BASE_H = 720`; `resize()` scales the canvas to fit the window (capped at 1.6×). Position and size values should be in base coordinates.
- **Time-based movement.** Update logic takes `dt` in seconds; do not tie movement to frame count, except for existing animation counters like `frame`.
- **Tuning lives in data tables.** Prefer adjusting `DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, and `THEMES` over hard-coding numbers in logic.
- **Accessibility and preferences.** Respect the reduced-motion setting (`reducedMotion`) when adding visual effects such as shake or flashes. Keep `announce()` / `srLive` updates for important state changes.
- **Input parity.** Any new keyboard action needs a touch equivalent in `touchControls` (the game supports phones; `viewport` disables user scaling).
- **Pause and game over.** New timers or effects must stop when `paused` or `gameOver` is set, and must reset in `resetGame()`.
- **Audio gating.** Call `ensureAudio()` from user-gesture handlers only; never start audio on load.
- **Keep the existing style:** two-space indentation, single quotes in JS, short helper functions, and comments only where the logic is non-obvious.

## Git

- Default development branch for automated work: `claude/gifted-tesla-4ffe9u`. Do not push to `main` without explicit permission.
- Commit messages: imperative summary line, then a short body describing what changed and why.
- Do not open pull requests unless explicitly asked.

## When changing the game

1. Read the relevant section before editing; the file is large (~1,670 lines), so use targeted reads/greps.
2. Make the smallest change that satisfies the request. Do not reformat unrelated code.
3. Check by playing in a browser: start, play a stage, pause/resume, use a bomb, lose a life, and game over. Confirm no console errors.
4. Report what you verified and what you could not verify.
