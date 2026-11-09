# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a small, dependency-free static web project. Its only real content is **Sky Fighter**, a Raiden-style vertical shoot-'em-up written as a single-page HTML5 canvas game.

| File | Purpose |
| --- | --- |
| `index.html` | The entire game: markup (`<body>`), inline CSS (`<style>`, ~lines 10–420) and inline JavaScript (`<script>`, starts ~line 422). |
| `README.md` | Minimal placeholder (title only). |

There is no build step, no package manager, no test suite, and no linter configured. Nothing needs to be installed.

## Running and verifying

- Open `index.html` directly in a browser, or serve the directory with any static server (e.g. `python3 -m http.server`) and visit it.
- There are no automated tests. Verify changes by playing the game in a browser, including the affected code path (keyboard, mouse, touch, pause, bombs, stage transitions, boss fight, game over) as relevant.
- For UI or gameplay changes, check both desktop and a narrow/mobile viewport: the canvas scales to fit the window and touch controls are shown on touch devices.

## Architecture of `index.html`

The game is wrapped in one IIFE (`(function () { ... })();`). Everything is module-private; nothing is exposed on `window`.

Key sections, in source order:

1. **DOM refs** – `document.getElementById(...)` lookups for every HUD element, overlay element, and touch control.
2. **Constants**
   - `STORAGE_*` keys (`skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion`).
   - `DIFFICULTIES` – `easy` / `normal` / `hard` tuning (spawn rates, enemy speed, shoot chance, starting lives and bombs, boss HP multiplier).
   - `KILLS_PER_STAGE` (18) – kills needed to trigger a boss.
   - `THEMES` – six stage background palettes; cycled per stage.
   - `ENEMY_TYPES` – `grunt`, `interceptor` (weaving), `tank` (4 HP, unlocks at stage 2), with `score` values.
   - `BOSSES` – six boss designs; `hpMul` scales their health.
3. **Persistence & settings** – best score, mute, difficulty, and reduced-motion are read from and written to `localStorage`. Every access should stay defensive (private mode and blocked storage can throw).
4. **Performance tracking** – `trackPerf` / `quality` (0 = full, 1 = medium, 2 = low) lowers visual effort on slow frames.
5. **Audio** – a lazily created `AudioContext` (`ensureAudio`) with `beep`, `noiseBurst`, and a named `sfx` table. Audio is only started after a user gesture.
6. **Rendering helpers** – `drawPlane`, `drawBossShip`, `drawClouds`, particles, and the main `draw()`.
7. **Entities & spawning** – `spawnEnemy`, `spawnBoss`, `spawnExplosion`, `useBomb`.
8. **Game loop** – `update(dt)` (game logic), `updateBoss(dt)`, `draw()`, and `loop(now)` driven by `requestAnimationFrame`.
9. **Input** – keyboard (`keys` map; arrows/WASD move, Space fires, X/Shift bomb, P/Esc pause), mouse (move to steer, click to fire on the canvas), and touch (joystick in `moveZone`, `fireBtn`, `bombBtn`).
10. **Lifecycle** – `resetGame` (fresh run state), `togglePause`, `endGame` (writes best score), and overlay/HUD updates.

Canvas is a fixed logical size of **480 × 720** (`BASE_W`, `BASE_H`). `resize()` scales the CSS size to fit the window (capped at 1.6×). Game coordinates should always be expressed in the 480 × 720 space, not in CSS pixels.

## Conventions

- **Single file, no dependencies.** Keep the game in `index.html`. Do not add a bundler, framework, or external script/CDN dependency unless the user explicitly asks for it.
- **Match the existing style.** 2-space indentation, single quotes in JS, `const`/`let` (no `var`), arrow functions for callbacks, and short section comments that match the existing ones.
- **Tuning lives in the constant tables.** Prefer changing `DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `THEMES`, or `KILLS_PER_STAGE` over hard-coding numbers inside update logic.
- **Frame-rate independence.** `update(dt)` receives a delta time; new movement or timers should use `dt`, not a fixed per-frame step.
- **Respect reduced motion.** Screen shake, flashes, and pulsing effects should honor the `reducedMotion` setting.
- **Preserve touch and keyboard parity.** Any new player action needs a keyboard binding and a touch control, and the `#controlsHint` text should be updated to match.
- **Keep the HUD compact.** HUD items are pill-style `.stat` elements in `#hud`; new stats should follow that pattern and remain readable at phone width.
- **Accessibility.** Announcements go through `announce()` (the `srLive` live region). Keep new UI keyboard-reachable where practical.

## Git and workflow

- Develop on the branch the session was given (currently `claude/gifted-tesla-hbyn6l`). Commit with clear messages describing the gameplay or UI change, and push with `git push -u origin <branch>`.
- Do not open a pull request unless explicitly asked.
- Before editing `index.html`, read the relevant section; the file is large (~1,670 lines) and most logic is in one IIFE, so small, targeted edits are safer than rewrites.

## Things to be careful about

- Game state (`score`, `lives`, `boss`, `stageKills`, etc.) is shared across the IIFE. Resetting a feature usually means touching `startGame`/`endGame` as well as the feature itself.
- `localStorage` writes happen on `endGame` (best score) and on setting changes. Changing a storage key will orphan existing players' saved data.
- Do not remove the `preventDefault` on Space; without it the page scrolls.
