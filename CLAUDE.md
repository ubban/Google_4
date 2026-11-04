# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo is **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs in the browser. The entire game is a single self-contained file, `index.html`, with inline CSS and one inline `<script>` block. There is no build step, no bundler, no package manager, no external dependencies, and no test suite.

- `index.html` — the whole game (markup, styles, and game logic).
- `README.md` — minimal placeholder.

To run the game, open `index.html` in a browser, or serve the directory with any static server.

## Layout of `index.html`

- **Markup (~lines 362–420):** `#container` holds the `<canvas id="gameCanvas">`, the HUD (`#hud`: score, best, multiplier, weapon, hearts, bombs), boss bar, combo/stage banners, top-bar buttons (mute, reduced-motion, pause), touch controls (joystick + FIRE/BOMB), and the `#overlay` start/game-over card with difficulty buttons.
- **Config constants (~lines 463–516):** `STORAGE_*` localStorage keys, `DIFFICULTIES`, `KILLS_PER_STAGE` (18), `THEMES` (six stage backdrops), `ENEMY_TYPES` (grunt, interceptor, tank), and `BOSSES` (six boss designs, cycled by stage).
- **Audio (~lines 543–680):** lazily created `AudioContext`; `beep`, `noiseBurst`, and the `sfx` table. Sound is muted by the mute toggle and persisted.
- **Input (~lines 768–870):** keyboard listeners, a virtual joystick and fire/bomb buttons for touch, and `canvasPointFromEvent` for pointer input.
- **Game state and flow:** `resetGame()` (~875), `update(dt)` (~1148, the main simulation step), `applyPowerUp` (~1304), boss attack patterns (`fireRadial`, `fireAimedSpread`, `fireSpiralBurst`, `fireDoubleRing`, etc.), `updateBoss`, `killBoss`, `loseLife`, `endGame`, and `togglePause`.
- **Rendering:** `draw()` (~1520) and helpers `drawPlane`, `drawBossShip`, `drawClouds`. `loop(now)` (~1640) drives the frame loop.

The game uses a fixed logical resolution of **480×720** (`BASE_W`, `BASE_H`), scaled to fit the window by `resize()` (capped at 1.6×).

## Gameplay reference

- **Controls:** Arrow keys or WASD move; Space fires; X or Shift drops a bomb; P or Escape pauses. Touch uses the left joystick and right-side FIRE/BOMB buttons.
- **Difficulty** (`DIFFICULTIES`): Easy, Normal, Hard. Each sets spawn rate, enemy speed, enemy shot chance, starting lives, starting bombs, and boss HP multiplier.
- **Stages:** clearing `KILLS_PER_STAGE` kills spawns a boss; defeating it advances `stage`, which changes the theme and boss design.
- **Power-ups** (`maybeSpawnPowerUp` / `applyPowerUp`): life, bomb, and weapon. Weapon pickups upgrade level 1→3, then toggle between `vulcan` and `laser`.
- **Persistence:** best score, mute, difficulty, and reduced-motion preference are stored in `localStorage` under `skyfighter.*` keys. Every access should stay wrapped so the game still works if storage is unavailable.
- **Accessibility:** respects `prefers-reduced-motion` (with a user toggle) for screen shake, flashes, and pulses; announces key events via the `#srLive` live region.

## Conventions

- Keep the game as a **single self-contained HTML file** unless the user explicitly asks to split it. Do not introduce a build system, frameworks, or CDN dependencies without asking.
- Match the existing style: vanilla JS, `const`/`let`, 2-space indentation, and short descriptive function names.
- Tuning values (speeds, spawn rates, HP, scores) live in the config constants or inline in the relevant function. Prefer changing those over restructuring logic.
- Preserve mobile behavior: touch controls, `touch-action: none`, safe-area insets, and the phone-width layout.
- Keep the `update` / `draw` split: simulation changes go in `update`, visuals in `draw`.

## Verifying changes

There are no automated tests or linters. To verify a change:

1. Open `index.html` in a browser (or run a static server such as `python3 -m http.server` from the repo root).
2. Start a game on each difficulty, and check that movement, firing, bombs, power-ups, stage transitions, boss fights, pause, and game over all still behave.
3. Check the touch controls and a narrow viewport if the change touches input or layout.
4. Confirm no errors appear in the browser console.

For visual or gameplay changes, prefer running the app and observing it over relying only on reading the code.

## Git

- Develop on the branch assigned to the session (currently `claude/gifted-tesla-d5d296`) and push with `git push -u origin <branch>`.
- Write clear, descriptive commit messages focused on the gameplay or code change.
- Do not open a pull request unless the user asks for one.
