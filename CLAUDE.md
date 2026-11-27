# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo contains **Sky Fighter**, a single-file, browser-based Raiden-style vertical shoot-'em-up. There is no build step, package manager, or framework.

- `index.html` — the entire game: inline `<style>`, markup, and one inline `<script>` (an IIFE starting near line 423) using an HTML5 `<canvas>` and the Web Audio API.
- `README.md` — minimal project readme (title only).

There is no test suite, linter, or CI configuration in the repo.

## Running the game

Open `index.html` directly in a browser, or serve the directory with any static server, e.g. `python3 -m http.server` and visit `http://localhost:8000`. No install step is needed.

## Code structure (inside `index.html`)

The script is one IIFE. Key areas, roughly in file order:

- **Constants** — `STORAGE_*` localStorage keys (`skyfighter.*`), `DIFFICULTIES` (spawn rates, enemy speed, shoot chance, lives, bombs, boss HP multiplier), `KILLS_PER_STAGE`, `THEMES`, `ENEMY_TYPES`, `BOSSES`, and the base logical canvas size `BASE_W × BASE_H` (480×720).
- **Persistent settings** — best score, mute, difficulty, and reduced-motion are read from and written to `localStorage`.
- **Audio** — `ensureAudio`, `beep`, `noiseBurst`, plus the `EXPLODE_SFX` mapping. Audio is created lazily on first user gesture.
- **HUD / effects** — `renderHearts`, `renderBombs`, `flashHit`, `flashBomb`, `showComboPop`, `showStageBanner`, `screenShake`. Visual effects respect the reduced-motion setting.
- **Input** — keyboard (`keys[e.code]`), mouse (canvas pointer steering and click-to-fire), and touch (virtual joystick on `#moveZone`, `#fireBtn`, `#bombBtn`).
- **Rendering** — `drawPlane`, `drawBossShip`, and related draw helpers.
- **Simulation** — `update(dt)` is the per-frame game step; `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`, `spawnExplosion`, `addScore`/combo logic, `useBomb`, `fireWeapon`, `applyPowerUp`, and the boss bullet patterns (`fireRadial`, `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`).
- **Game state** — `resetGame()` reinitialises `player` and the other run state. Pause is toggled with `togglePause()`.

### Controls (for reference when editing input code)

- Keyboard: arrows/WASD move, Space fire, X or Shift bomb, P or Esc pause.
- Mouse: move to steer, click to fire.
- Touch: drag left zone to fly, FIRE and BOMB buttons.

## Conventions

- Keep the game in a single self-contained `index.html` unless the user asks to split it. Do not add a bundler, framework, or npm dependencies without explicit approval.
- Keep the existing style: `const` / `let`, two-space indentation, and the `STORAGE_*` key prefix `skyfighter.` for any new persisted setting.
- Put tunable numbers in the constant blocks (e.g. `DIFFICULTIES`) rather than hard-coding them inside `update()` or spawn functions.
- Any new HUD element or animation should honour the reduced-motion setting, and any new interactive control needs an accessible label or `title`, as existing buttons do.
- Layout must work on phones: keep the `viewport-fit=cover` / safe-area handling and the no-scroll `html, body` styling.

## Verifying changes

Because there are no automated tests, verify changes by loading `index.html` in a browser and playing through: start, fire, bomb, a stage boss, pause/resume, game over, and difficulty switching. For touch changes, check with a touch-emulated viewport. Run the `run` skill if a full app launch check is needed.

## Git

- Default development branch for this session: `claude/gifted-tesla-52t4s3`. Push with `git push -u origin <branch>`.
- Do not create pull requests unless explicitly asked.
