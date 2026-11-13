# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Google_4 is a small static web project. Its only real content is **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs in a single HTML file, `index.html`, drawn on a `<canvas>`.

Files:

- `index.html` — the entire game: inline `<style>`, markup, and one inline `<script>` IIFE (`'use strict'`).
- `README.md` — a placeholder title; no real documentation yet.

There is no build step, no package manager, no bundler, no test suite, and no linter configuration. Do not add any of these unless asked.

## Running the game

Open `index.html` directly in a browser, or serve the directory with any static server (for example `python3 -m http.server`) and browse to it. No install step is needed.

## Code layout inside `index.html`

- **Markup/CSS** (top of file): `#container`, `#gameCanvas`, HUD elements (`#hud`, `#hearts`, `#bombs`, `#weaponStat`, `#multStat`), `#touchControls` (joystick + FIRE/BOMB buttons), and `#overlay` (title, difficulty buttons, Start button, control hints). The viewport meta disables user scaling, and `touch-action: none` is set on the body.
- **Constants**:
  - `BASE_W = 480`, `BASE_H = 720` — the logical play-field size; `resize()` scales the canvas to fit the window.
  - `DIFFICULTIES` — per-difficulty spawn rates, enemy speed, shoot chance, starting lives/bombs, and boss HP multiplier.
  - `KILLS_PER_STAGE` — kills needed before a stage advances.
  - `THEMES` — sky gradient palettes cycled per stage.
  - `ENEMY_TYPES` (`grunt`, `interceptor`, `tank`) and `BOSSES` (six bosses with colors and HP multipliers).
  - `STORAGE_*` keys — all persistence uses `localStorage` under the `skyfighter.*` prefix (best score, mute, difficulty, reduced motion).
- **Audio**: a lazily created `AudioContext` (`ensureAudio()`), with `beep()` and `noiseBurst()` for synthesized sound effects. Audio starts only after a user gesture.
- **Game state and loop**: `resetGame()` initializes state; `update(dt)` advances the simulation; `draw()` renders; `loop(now)` is the `requestAnimationFrame` driver. `togglePause()` and `endGame()` handle the run lifecycle.
- **Input**: keyboard (`keydown`/`keyup` on `window`, arrows/WASD, Space to fire, X or Shift for bomb, P/Esc to pause), mouse (move to steer, click to fire on the canvas), and touch (`joystickStart/Move/End` for the left-side drag zone, plus FIRE and BOMB buttons).
- **Bosses**: `spawnBoss()`, `BOSS_PATTERNS` (bullet-pattern functions such as `fireRadial`, `fireSpiralBurst`, `fireHomingSeeded`), `updateBoss()`, `killBoss()`.
- **Accessibility**: `announce()` writes to the `#srLive` aria-live region. A reduced-motion mode (`reducedMotion`, toggled by `#motionBtn`) disables screen shake and flashes. The initial value follows `prefers-reduced-motion` unless the user has set it.

## Conventions

- Keep the game in a single self-contained file unless the user asks for a split. Do not introduce external dependencies or CDN scripts.
- Keep the code inside the existing IIFE. Do not add new globals.
- Keep the style of the surrounding code: 2-space indentation, `const`/`let`, arrow functions for callbacks, and single quotes in JavaScript.
- Preserve the mobile layout: the viewport must fit a phone screen, and the touch controls must remain usable.
- Any change that affects game balance should go through the `DIFFICULTIES`, `ENEMY_TYPES`, or `BOSSES` constants rather than hard-coded numbers in the update loop.
- Respect the `localStorage` keys listed above. Renaming a key silently resets a player's saved best score, mute, difficulty, or motion preference, so treat renames as breaking changes.

## Testing and verification

There is no automated test suite. To verify a change:

1. Open `index.html` in a browser (or a static server) and check the browser console for errors.
2. Start a game on each difficulty, and check steering, firing, bombs, pausing, stage transitions, boss fights, and game over.
3. Check the mobile layout in a narrow viewport, and confirm the touch controls still work.
4. Confirm the best score persists across reloads.

If you add tests later, document the command here.

## Git

- Develop on the branch assigned for the session, and push with `git push -u origin <branch>`.
- Write commit messages in the imperative mood with a short subject line.
