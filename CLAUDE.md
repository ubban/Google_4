# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Google_4 is a small static web project. Its only real content is **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs in a single browser page.

- `index.html` — the entire game: markup, CSS (`<style>`, ~lines 10–359), and JavaScript (`<script>`, ~lines 422–1667). No external assets, libraries, or network calls.
- `README.md` — a one-line placeholder. Keep it accurate if you change how the project is described.

There is no build step, bundler, package manager, test suite, linter, or CI config. Do not add any of these unless asked.

## Running and verifying

- Open `index.html` directly in a browser, or serve the directory with any static server (e.g. `python3 -m http.server`).
- There are no automated tests. Verify changes by playing the game in a browser: start a run, check the affected behavior, and check both desktop (keyboard/mouse) and touch layouts if the change touches input or HUD.
- Keep the game working when opened from `file://`. Do not introduce features that require a server.

## Code structure inside `index.html`

The script is one IIFE (`(function () { 'use strict'; ... })();`). Within it, the main areas are:

- **DOM references and storage keys** — top of the script. `STORAGE_*` constants are the `localStorage` keys (`skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion`). Keep these names stable so saved player data is not lost.
- **Tuning tables** — `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES` (one per stage, cycled), `ENEMY_TYPES`, `BOSSES`. Balance changes usually belong here, not in the update logic.
- **Audio** — `ensureAudio`, `beep`, `noiseBurst`, and the `EXPLODE_SFX` map. Audio is created lazily on the first user gesture, which browsers require.
- **Input** — keyboard (`keys`, `keydown`/`keyup`), touch joystick and buttons, and mouse steering on the canvas.
- **Game state and flow** — `resetGame`, `update(dt)`, `togglePause`, `applyPowerUp`, `useBomb`, `fireWeapon`, `addScore`/`breakChain`.
- **Enemy and boss patterns** — `spawnEnemy`, `spawnBoss`, the `fire*` pattern helpers, and `BOSS_PATTERNS`.
- **Rendering** — `draw()` and helpers such as `drawPlane`, `drawBossShip`, `drawClouds`. Drawing is canvas 2D only.
- **Main loop** — `loop(now)` drives `update` and `draw` each frame with a delta time.
- **Adaptive quality** — `trackPerf` lowers the `quality` level when frame time is high and raises it when frames are fast. Respect this when adding costly effects.
- **Accessibility** — `announce()` writes to a live region; `reducedMotion` (user setting, defaulting to the OS preference) should suppress or soften screen shake, flashes, and similar motion effects.

## Conventions

- **Single file.** Keep HTML, CSS, and JS in `index.html` unless the user asks for a split. Do not add a build step to split it.
- **Plain JavaScript and CSS.** No frameworks, no transpilation, no dependencies. Use ES2017+ syntax already present in the file (`const`/`let`, arrow functions, template literals).
- **Match the existing style.** Two-space indentation, single quotes in JS, `const` by default, and small named helper functions inside the IIFE. Do not add new globals.
- **Stay inside the IIFE.** Do not expose game internals on `window`.
- **Preserve persisted data.** Changing a `STORAGE_*` key or the meaning of a stored value resets players' best score, mute, difficulty, or motion settings. Only do this deliberately and mention it in the summary.
- **Frame-rate independence.** Movement and timers use the `dt` passed to `update`. Do not hard-code per-frame increments.
- **Mobile and desktop.** Any new control must work with both mouse/keyboard and touch. Keep the `BASE_W` × `BASE_H` (480×720) logical resolution and the `resize()` scaling in mind when changing layout.
- **Avoid sound on load.** Audio must start only after a user gesture.

## Git and workflow

- Develop on the branch designated for this session. Do not push to other branches without explicit permission.
- Commit only when asked. Keep commits focused on one change.
- Do not open a pull request unless the user explicitly asks for one.

## When making changes

1. Read the relevant section of `index.html` before editing; the file is large, so search for the function or constant first.
2. Make the smallest change that meets the request. Do not refactor unrelated code.
3. Test in a browser, including the affected input path and a full stage transition if gameplay logic changed.
4. Report what was changed and what was verified, and state plainly anything that was not tested.
