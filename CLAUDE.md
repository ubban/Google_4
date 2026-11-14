# CLAUDE.md

Guidance for Claude Code when working in this repository.

## What this project is

`Google_4` is a single-page browser game, **Sky Fighter**: a Raiden-style vertical shoot-'em-up with weapon switching, smart bombs, power-ups, combo multipliers, stages with bosses, and difficulty levels.

The entire game lives in one file:

- `index.html` (~1,670 lines): inline `<style>`, inline `<script>`, and the canvas + HUD/overlay markup. No build step, no bundler, no framework, no external dependencies or CDN links.
- `README.md`: title only.

There is no package manager, test suite, linter, or CI configuration in the repo.

## Running and verifying

- Open `index.html` directly in a browser, or serve the directory with any static server (e.g. `python3 -m http.server`).
- There are no automated tests. Verify changes by playing the game in a browser: start a run, fire, use a bomb, pause/resume, lose a life, clear a stage boss, and reach the game-over screen.
- For touch behavior, test with a touch device or browser device emulation. The touch joystick and fire/bomb buttons only appear when a touch device is detected.

## Code layout inside `index.html`

The script is organized roughly in this order. Search for the function name to find it:

| Area | Key identifiers |
| --- | --- |
| Persistence keys | `STORAGE_BEST`, `STORAGE_MUTE`, `STORAGE_DIFF`, `STORAGE_REDUCED` (all `skyfighter.*` in `localStorage`) |
| Difficulty | `DIFFICULTIES` (spawn rate, enemy speed, shoot chance, lives, bombs, boss HP multiplier), `setDifficulty` |
| Stages | `KILLS_PER_STAGE`, `THEMES` (sky palettes per stage), `BOSSES` |
| Enemies | `ENEMY_TYPES`, `spawnEnemy`, `spawnBoss`, `BOSS_PATTERNS` and the `fire*` pattern helpers |
| Audio | `ensureAudio`, `beep`, `noiseBurst`, `EXPLODE_SFX` (Web Audio, synthesized, no audio files) |
| Input | `keys` map, keyboard listeners, joystick (`joystick*`), `canvasPointFromEvent` |
| Game state | `resetGame`, `update(dt)`, `applyPowerUp`, `useBomb`, `fireWeapon`, `loseLife`, `endGame`, `togglePause` |
| Rendering | `draw`, `drawPlane`, `drawBossShip`, `drawClouds`, `spawnExplosion`, `screenShake` |
| HUD / UI | `renderHearts`, `renderBombs`, `showStageBanner`, `showComboPop`, `flashHit`, `announce` (screen-reader live region) |
| Main loop | `loop(now)` drives `update` and `draw` via `requestAnimationFrame`; `resize` handles canvas sizing (base logical size `BASE_W` x `BASE_H` = 480 x 720) |

### Controls

- Move: Arrow keys or WASD (`keys` map, handled in `update`).
- Mouse: move the pointer over the canvas to steer (`mouseTarget`); mousedown on the canvas fires.
- Fire: Space (also held via the FIRE button on touch).
- Bomb: `X` or either Shift key (also the BOMB button on touch).
- Pause: `P` or `Escape`.
- Touch: drag in the left zone to fly (virtual joystick), tap FIRE, tap BOMB.

If you change a control, update the keyboard/mouse/touch handlers in the "Input" section and the on-screen `controlsHint` text (`#controlsHint`).

## Conventions

- **Keep it a single file.** Do not split `index.html` into separate JS/CSS files or add a build step unless the user explicitly asks for that restructuring.
- **No external dependencies.** Do not add CDN scripts, fonts, or npm packages. The game uses only browser APIs (Canvas, Web Audio, `localStorage`, Pointer/Touch events).
- **Match the existing style.** Two-space indentation inside the `<script>` and `<style>` blocks, `const`/`let`, plain functions, and the `UPPER_SNAKE` naming used for tuning constants and storage keys.
- **Tuning lives in constants.** Difficulty numbers, stage counts, and boss patterns are data tables near the top of the script; prefer changing those over hard-coding values in logic.
- **Frame-rate independence.** `update(dt)` takes a delta time; scale movement and timers by `dt` rather than assuming 60 FPS.
- **Respect accessibility settings.** The game has a reduced-motion option (`reducedMotion`, `updateMotionBtn`) that gates screen shake, flashes, and heart-beat animation. New visual effects that move or flash should honor it. Announcements for screen readers go through `announce()`.
- **Persist only through the existing keys.** Store new player settings under `skyfighter.*` in `localStorage`, and wrap reads so that a missing or corrupt value falls back to a default.
- **Mobile-first layout.** The UI must work at phone width. The `#container`, `#hud`, and `#gameCanvas` rules use `clamp()`, `env(safe-area-inset-*)`, and `max-width: 100%`; keep new HUD elements responsive and avoid horizontal scrolling.

## Git

- Development branch for this work: `claude/gifted-tesla-pklyow`. Commit and push there. Do not push to other branches without explicit permission.
- Commit messages are short imperative summaries of the gameplay or code change (see the existing history: "Add Sky Fighter: Raiden-style vertical shoot-em-up game").
- Do not open a pull request unless the user asks for one.

## When making changes

1. Read the relevant section of `index.html` before editing; the file is large, so search for identifiers rather than reading it top to bottom.
2. Make the smallest change that satisfies the request. Do not reformat unrelated code.
3. Load `index.html` in a browser and exercise the affected feature, at both desktop and phone-width viewports, before reporting it as done.
4. In the final summary, state what was verified in a browser and what was not.
