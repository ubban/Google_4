# CLAUDE.md

Guidance for Claude Code when working in this repository.

## What this repo is

`Google_4` is a small static web project containing **Sky Fighter**, a Raiden-style vertical shoot-'em-up played on an HTML `<canvas>`.

- `index.html` — the entire game: markup, CSS (`<style>`), and JavaScript (`<script>`) in one file (~58 KB, about 1,670 lines).
- `README.md` — a one-line title. Keep it minimal unless the user asks for more.

There is no build step, no package manager, no test suite, no linter, and no external runtime dependencies. Do not introduce any of these unless the user asks.

## Running and verifying

- Open `index.html` directly in a browser, or serve the directory with any static server (for example `python3 -m http.server`).
- Verification is manual: load the page, play through the controls, and check the behavior you changed. If a headless browser is available, a Playwright smoke test (load page, start game, check for console errors) is a reasonable check. Do not add test tooling to the repo without being asked.
- Check the browser console for errors after any JS change.

## Code layout inside `index.html`

The script is wrapped in an IIFE and organized roughly in this order:

1. **Persistence keys** — `STORAGE_BEST`, `STORAGE_MUTE`, `STORAGE_DIFF`, `STORAGE_REDUCED` (all prefixed `skyfighter.`).
2. **Config constants** — `DIFFICULTIES` (spawn rates, enemy speed/fire chance, lives, bombs, boss HP multiplier), `KILLS_PER_STAGE`, `THEMES` (per-stage sky palettes), `ENEMY_TYPES` (`grunt`, `interceptor`, `tank`, with `minStage` gating), `BOSSES` (boss designs and HP multipliers).
3. **Audio** — lazily created `AudioContext` (`ensureAudio`), with `beep` and `noiseBurst` helpers. Audio is only started from a user gesture.
4. **UI helpers** — HUD rendering (hearts, bombs), flashes, combo pop, stage banner, screen shake, `announce()` for screen readers.
5. **Perf scaling** — `trackPerf` adjusts `quality` (0 full, 1 medium, 2 low) based on frame time.
6. **Input** — keyboard (`keys[e.code]`), a touch joystick (`joystickStart/Move/End`), on-screen fire and bomb buttons, and mouse/pointer aiming on the canvas.
7. **Game state** — globals such as `score`, `lives`, `stage`, `kills`, `chain`, `multiplier`, `player`, plus `resetGame()`.
8. **Entities and combat** — `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`, `fireWeapon`, `useBomb`, `applyPowerUp`, and the boss attack patterns (`fireRadial`, `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`, `fireCross`, `fireHomingSeeded`, `fireDoubleRing`).
9. **Update and draw** — `update(dt)` for simulation, `draw*` functions for rendering (`drawPlane`, `drawBossShip`, and others).
10. **Main loop** — `loop(now)` driven by `requestAnimationFrame`, with a clamped `dt`.

Keep new code in the matching section. Prefer small, named functions over inline logic in `update` or `loop`.

## Game rules worth knowing before editing

- **Controls:** arrows/WASD move (via `keys`), Space fires, `X` / Shift drops a smart bomb, `P` / Escape pauses. Touch and mouse controls mirror these.
- **Weapons:** two weapons, `vulcan` and `laser`, toggled by a weapon power-up (`applyPowerUp`). The fire delay differs by weapon (`0.15` vs `0.1`).
- **Power-ups** dropped from enemies: `life`, `bomb`, `weapon`.
- **Stages:** each stage is `KILLS_PER_STAGE` (18) kills, after which a boss fight is triggered. The theme and boss are picked by `(stage - 1) % length` from `THEMES` and `BOSSES`, so both cycle.
- **Difficulty** (`easy` / `normal` / `hard`) is stored in localStorage and changes starting lives, bombs, and spawn/enemy tuning.
- **High score** is saved to `skyfighter.bestScore` in localStorage when a game ends.
- **Reduced motion** is a user setting (`body.reduced-motion`) that disables or softens animations, shake, and banners. Any new animation should respect it.

## Conventions

- Keep everything in `index.html` unless the user asks to split it. Splitting would change how the project is served, so confirm first.
- Match the existing style: 2-space indentation, single quotes in JS, `const` by default, `let` when reassigned, and short section comments.
- Do not add external CDN scripts, fonts, or frameworks. The game is self-contained by design.
- Every new user-visible control needs a keyboard path, a touch/mouse path where relevant, and an `aria-label` or `announce()` call.
- When adding a new config value, put it in the relevant constant table (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `THEMES`) instead of hard-coding it in logic.
- Persist new settings through a `skyfighter.*` localStorage key and wrap storage access so the game still works if storage is unavailable.

## Git

- Development branch for this work: `claude/gifted-tesla-4cpcig`. Push there with `git push -u origin <branch>`.
- Do not open a pull request unless the user asks for one.
- Commit messages: imperative summary line, then a short body if useful. Describe the gameplay or code change, not the process.
