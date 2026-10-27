# CLAUDE.md

Guidance for working in this repository.

## Project overview

Google_4 is a small static web project. Its only real content is **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs in a single HTML5 canvas page.

| File | Purpose |
| --- | --- |
| `index.html` | The whole game: markup, CSS, and one inline `<script>` IIFE (~1,670 lines). |
| `README.md` | Placeholder project title. |

There is no build step, no package manager, no test suite, and no external dependencies. Everything is vanilla HTML/CSS/JS.

## Running the game

Open `index.html` directly in a browser, or serve the folder with any static server, for example:

```bash
python3 -m http.server 8000
```

## Code layout inside `index.html`

- **Lines 10–359**: inline `<style>` (HUD pills, overlay card, touch controls, reduced-motion and low-quality styles).
- **Lines 361–421**: DOM markup (`#gameCanvas`, `#overlay`, `#hearts`, `#bombs`, `#touchControls`, and so on).
- **Lines 422 onward**: one `<script>` wrapped in `(function () { 'use strict'; ... })()`. Nothing is exposed on `window`.

Inside the script, the code is grouped by section comments:

1. **DOM references and storage keys** — `STORAGE_*` constants prefixed `skyfighter.`.
2. **Adaptive performance** — `trackPerf` samples frame time and steps a `quality` level (0–2) up or down.
3. **Config tables** — `DIFFICULTIES`, `THEMES` (sky palettes per stage), `ENEMY_TYPES`, `BOSSES`, `KILLS_PER_STAGE`.
4. **Audio** — lazy `ensureAudio()` with WebAudio `beep()` and `noiseBurst()`. Audio is created only after a user gesture.
5. **HUD helpers** — hearts, bombs, combo pop, stage banner, screen shake, flash.
6. **Layout and input** — `resize()` scales to a fixed `BASE_W × BASE_H` (480 × 720) logical canvas. Keyboard, pointer, and touch joystick/fire/bomb handlers.
7. **Game state** — `resetGame()`, `update(dt)`, `applyPowerUp()`, `useBomb()`, `fireWeapon()`, `spawnEnemy()`, `spawnBoss()`.
8. **Boss patterns** — `BOSS_PATTERNS` array of bullet generators (`fireRadial`, `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`, `fireCross`, `fireHomingSeeded`, `fireDoubleRing`) driven by `updateBoss()`.
9. **Rendering** — `draw()`, plus `drawPlane()`, `drawBossShip()`, `drawClouds()`.
10. **Loop** — `loop(now)` clamps `dt` to 50 ms, then calls `update` and `draw` only when running and not paused.

## Game rules and conventions

- **Controls.** Arrow keys or WASD move. Space fires. `X` or Shift drops a bomb. `P` or Escape pauses. Touch devices use a joystick zone plus fire and bomb buttons.
- **Difficulty.** `easy`, `normal`, and `hard` change spawn rate, enemy speed, enemy fire chance, starting lives, starting bombs, and boss HP multiplier. Adjust balance in `DIFFICULTIES`.
- **Progression.** Each stage lasts `KILLS_PER_STAGE` kills (18), then a boss fight starts. Stage and theme cycle through `THEMES` and `BOSSES`.
- **Persistence.** Only four `localStorage` keys are used: best score, mute, difficulty, and reduced-motion. Every access must stay inside the `skyfighter.*` namespace.
- **Accessibility.** Respect `prefers-reduced-motion` (the `reducedMotion` flag and the `reduced-motion` body class). Announce HUD messages through the `#srLive` live region via `announce()`.
- **Scoring.** Score uses a kill-chain multiplier (`breakChain`, `addScore`). Combos show as `#comboPop`.

## Editing guidelines

- Keep everything in `index.html` unless there is a clear reason to split it. The single-file design is intentional and makes the project easy to host.
- Match the existing style: 2-space indentation, single quotes in JS, `const` by default, short section comments using the `// ---------- Name ----------` pattern.
- When adding an enemy, boss, or pattern, add it to the config table first, then wire the behavior into `spawnEnemy`, `updateBoss`, or `BOSS_PATTERNS`.
- Keep the game loop allocation-light. Reuse arrays and avoid creating new objects per frame in `update` and `draw` where it is practical.
- Do not add external CDN scripts or fonts without a reason. The game is meant to run offline.

## Verification

There are no automated tests or linters. To check a change:

1. Load `index.html` in a browser and confirm there are no console errors.
2. Play through at least one stage and one boss on `normal`.
3. Check pause (`P`), bomb (`X`), mute, and difficulty persistence after a reload.
4. For layout changes, check a narrow viewport (phone width) and the touch controls.

## Git

- Develop on the branch designated for the task (currently `claude/gifted-tesla-0135p7`). Do not push to `main` without explicit permission.
- Write clear, descriptive commit messages in the imperative mood.
