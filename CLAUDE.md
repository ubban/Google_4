# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo contains **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs in the browser on an HTML5 `<canvas>`.

- `index.html` — the entire game: markup (~lines 1–420), CSS (inline `<style>`, lines ~10–359), and JavaScript (inline `<script>`, lines ~422–1669).
- `README.md` — minimal project readme.

There is no build step, no package manager, no bundler, no test suite, and no external dependencies. Everything is vanilla HTML/CSS/JS.

## Running the game

Open `index.html` directly in a browser, or serve the folder with any static server, for example:

```bash
python3 -m http.server 8000
```

Then visit `http://localhost:8000/`.

## Architecture of `index.html`

The script is one IIFE (`'use strict'`) with these sections, in order:

1. **DOM lookups** — every element is fetched once by id at the top. Element ids must stay in sync with the markup.
2. **Persistence** — `localStorage` keys prefixed `skyfighter.`: `bestScore`, `muted`, `difficulty`, `reducedMotion`. Reads and writes should tolerate missing or blocked storage.
3. **Adaptive performance** — `trackPerf` samples frame times and steps a `quality` level (0 full, 1 medium, 2 low) that throttles effects.
4. **Config tables**
   - `DIFFICULTIES` — `easy` / `normal` / `hard`, controlling spawn rate, enemy speed, enemy fire chance, starting lives and bombs, and boss HP multiplier.
   - `KILLS_PER_STAGE` (18) — kills needed to advance a stage.
   - `THEMES` — six sky palettes cycled per stage.
   - `ENEMY_TYPES` — `grunt`, `interceptor` (weaving), `tank` (4 HP, unlocks at stage 2).
   - `BOSSES` — six bosses (VANGUARD through TITAN), each with colors and an `hpMul`.
   - `BOSS_PATTERNS` — bullet-pattern functions used by bosses (radial, spread, spiral, cross, homing, double ring, etc.).
5. **Audio** — a lazily created Web Audio context (`ensureAudio`) with `beep` / `noiseBurst` helpers and the `sfx` table. Audio is only created after a user gesture.
6. **Input** — keyboard, touch joystick (`moveZone`), fire and bomb buttons, and mouse. Touch controls are shown only on touch devices.
7. **Game state** — `resetGame()` initialises the `player`, `bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`, and `boss` state. `update(dt)` advances the simulation, and `draw()` renders it.
8. **Main loop** — `loop(now)` caps `dt` at 0.05s, calls `trackPerf`, then `update` and `draw` when running and not paused. It is started once with `requestAnimationFrame`.

### Controls (keep README/overlay copy in sync if changed)

- Move: arrow keys or WASD; mouse (steer by moving over the canvas); or the on-screen joystick on touch devices.
- Fire: Space (hold); mouse button (hold); or the fire button on touch devices.
- Bomb: X or Shift; or the bomb button.
- Pause: P or Escape.

### Power-ups

Dropped power-ups are `life`, `bomb`, and `weapon` (`maybeSpawnPowerUp`). Weapons are `vulcan` and `laser`, each with a `level`.

## Conventions

- **Single file.** Keep the game in `index.html` unless the user asks for a split. Do not introduce a build tool, framework, or CDN dependency without an explicit request.
- **Vanilla JS, ES2017+ style.** Match the surrounding code: `const`/`let`, arrow functions, template literals where helpful, and no semicolon-less style.
- **Frame-rate independence.** Movement and timers should use `dt` (seconds), not per-frame constants.
- **Respect the quality and reduced-motion settings.** New particle or screen-shake effects should consult `quality` and `reducedMotion`.
- **Accessibility.** Status messages go through `announce()` to the `srLive` live region. Buttons keep their labels, and the reduced-motion toggle must keep working.
- **Storage access must be defensive.** Wrap new `localStorage` usage so the game still runs when storage is unavailable.
- **Keep the pause and game-over flows intact** when changing `update`, `endGame`, `loseLife`, or `togglePause`.

## Making changes

- When tuning balance, change the values in `DIFFICULTIES`, `ENEMY_TYPES`, or `BOSSES` rather than hard-coding numbers in the update loop.
- When adding a boss pattern, add a function next to the existing ones and register it in `BOSS_PATTERNS`.
- When adding a theme, append to `THEMES` with `sky`, `cloud`, `sun`, and `name`.
- After any change, open the game in a browser and check: start, play through a stage transition, lose a life, use a bomb, pause and resume, and game over with a best-score update.

## Verification

There is no automated test suite or linter configured. Verify changes by:

1. Loading `index.html` in a browser with no console errors.
2. Exercising the controls described above (keyboard and, if relevant, touch).
3. Checking that the HUD (score, multiplier, weapon, hearts, bombs, stage banner, boss bar) updates correctly.

If a headless browser is needed, Playwright with the pre-installed Chromium works well for this kind of check.

## Git

- Development branch for this session: `claude/gifted-tesla-sktint`.
- Commit messages: short imperative summary line, then an optional body explaining the why.
