# CLAUDE.md

Guidance for working in this repository.

## Project overview

Google_4 contains **Sky Fighter**, a Raiden-style vertical shoot-em-up built as a single-page HTML5 canvas game.

- `index.html` — the entire game: markup, CSS (inline `<style>`), and JavaScript (inline `<script>`).
- `README.md` — placeholder title only.

There is no build step, no package manager, no bundler, no test suite, and no linter configuration. Do not add tooling unless the user asks for it.

## Running the game

Open `index.html` directly in a browser. No server is required.

To verify a change, load the page and play through the affected behavior. Check the browser console for errors. Because the game is canvas-driven, visual changes need a manual check in the browser.

## Architecture of `index.html`

The script is one IIFE with `'use strict'`. Everything lives in its closure; nothing is exported to `window`. Keep new code inside that closure.

The JavaScript is organized into commented sections, using banner comments such as `// ---------- Adaptive performance ----------`. Follow that pattern when adding a section.

Key areas, roughly in file order:

- **DOM references** — `const` lookups for every element the game touches, grouped at the top.
- **Persistence keys** — `STORAGE_*` constants under the `skyfighter.*` namespace (best score, mute, difficulty, reduced motion).
- **Adaptive performance** — `trackPerf` samples frame times and steps a `quality` level (0–2) up or down.
- **Tuning tables** — `DIFFICULTIES`, `THEMES`, `ENEMY_TYPES`, `BOSSES`, and `KILLS_PER_STAGE`. Balance changes usually belong here, not in the update logic.
- **Audio** — `ensureAudio`, `beep`, `noiseBurst`. Audio is created lazily on the first user gesture.
- **Input** — keyboard (`keys[e.code]`), touch joystick and buttons, and mouse. Pause is bound to `KeyP`/`Escape`; bombs to `KeyX`/`Shift`.
- **Game state** — `resetGame()` initializes the state. Key state includes `score`, `lives`, `maxLives`, `running`, `paused`, and `gameOver`.
- **Simulation** — `update(dt)` advances entities. Boss attacks are defined as functions in `BOSS_PATTERNS`, and `updateBoss` / `killBoss` manage boss phases.
- **Rendering** — `draw()` renders each frame. `drawPlane`, `drawBossShip`, and `drawClouds` are the helpers.
- **Loop** — `loop(now)` clamps `dt` to 0.05s, then calls `update` and `draw` when the game is running and not paused.

### Conventions

- Time-based motion uses `dt` in seconds. Do not tie movement to frame count.
- Visual effects that shake or flash must respect `reducedMotion`.
- Announce state changes for screen readers through `announce()`, which writes to the `srLive` region.
- Persist settings through `localStorage` with the existing `STORAGE_*` keys. The current reads and writes are not wrapped in try/catch; wrap any new ones, since storage can be blocked in private windows or with site data disabled.
- Match the existing style: 2-space indent, single quotes, `const` by default, and short comments that explain why rather than what.

## Making changes

- Keep the game a single self-contained HTML file unless the user asks for a split.
- Prefer small, focused edits. Do not reformat unrelated code.
- Balance tweaks go in the tuning tables first.
- Test in a desktop browser and, for input changes, with touch emulation. The touch controls are enabled only when `isTouchDevice` is true.

## Git

- Develop on the branch the session specifies. The current working branch is `claude/gifted-tesla-amhdwi`.
- Write clear, descriptive commit messages.
- Do not open a pull request unless the user asks for one.
