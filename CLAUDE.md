# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Google_4 is a small static web project. Its only game is **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs in the browser on an HTML5 `<canvas>`.

- `index.html` — the whole game: markup, inline CSS, and an inline `<script>` (one IIFE, roughly lines 423–1665).
- `README.md` — minimal project readme.

There is no package manager, build step, bundler, test suite, or linter. Do not add one unless asked.

## Running and verifying

- Open `index.html` directly in a browser, or serve the directory with any static server (e.g. `python3 -m http.server`).
- Verify changes by playing the game in a browser, not just by reading the code. Check keyboard, mouse, and touch controls, and check the layout at phone width.
- Keep the page working with no network access. Everything is inline; do not add CDN scripts or fonts without a reason.

## Code conventions

- Keep the game in `index.html` unless the user asks for a split. Preserve the single-file, dependency-free design.
- Game tuning lives in named constants near the top of the script: `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES`, `ENEMY_TYPES`, `BOSSES`, and `BOSS_PATTERNS`. Change balance there rather than in the update loop.
- State is module-level `let` variables inside the IIFE (score, lives, `running`, `paused`, `gameOver`, and so on). Reset them in `resetGame()`. Do not leak new globals to `window`.
- The main loop is `update(dt)` followed by `draw()`, driven by `requestAnimationFrame` in `loop(now)`. Put logic in `update` and rendering in `draw`.
- Persisted settings use `localStorage` keys prefixed `skyfighter.` (`bestScore`, `muted`, `difficulty`, `reducedMotion`). Wrap storage access so the game still runs if storage is unavailable (private browsing, blocked storage).
- Respect the `reducedMotion` setting when adding screen shake, flashes, or other motion effects.
- Keep the UI usable on touch devices: the existing layout uses a left joystick zone, a FIRE button, and a BOMB button.

## Controls (for reference when editing input)

- Keyboard: Arrows or WASD to move, Space to fire, X or Shift to use a bomb, P or Esc to pause.
- Mouse: move to steer, hold to fire.
- Touch: drag in the left zone to fly, tap FIRE, tap BOMB.

## Game-design reference

- Difficulty (`easy` / `normal` / `hard`) sets lives, starting bombs, spawn rate, enemy shooting, and boss HP multiplier.
- Enemies: `grunt`, `interceptor` (weaving), and `tank` (4 HP, from stage 2).
- Power-ups: life, bomb, and weapon upgrades (weapon level up to 3; `vulcan` spreads more shots at higher levels).
- Stages advance every `KILLS_PER_STAGE` kills, cycling through `THEMES` and `BOSSES`.

## Working in this repo

- Develop on the branch the session specifies. Do not push to other branches without permission.
- Keep commits focused on one change, with a message that says what changed and why.
- Do not create pull requests unless the user asks for one.
- Treat `README.md` as the place for user-facing setup notes. Keep this file for instructions to Claude.
