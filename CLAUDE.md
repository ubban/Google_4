# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Google_4 is a small static web game, **Sky Fighter**: a Raiden-style vertical shoot-'em-up. The entire game lives in a single self-contained file, `index.html` (inline CSS + one inline `<script>` IIFE, about 1,670 lines, no build step, no dependencies). `README.md` is a placeholder.

There is no package manager, no build, no test suite and no linter configured. To run the game, open `index.html` in a browser (or serve the directory with any static server, e.g. `python3 -m http.server`).

## Repository layout

- `index.html` — the whole game: markup (`#container`, `#hud`, `#topBar`, `#touchControls`, `#overlay`), styles (lines ~10–361), and game logic (inline script starting ~line 422).
- `README.md` — minimal placeholder.

## Architecture of `index.html`

The script is wrapped in a single IIFE. Major sections, in source order:

- **DOM handles** — `getElementById` lookups at the top of the script.
- **Storage keys** — `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion` (via `localStorage`).
- **Tuning tables** — `DIFFICULTIES` (easy/normal/hard: spawn rates, enemy speed, shoot chance, lives, bombs, boss HP multiplier), `KILLS_PER_STAGE`, `THEMES` (per-stage sky palettes), `ENEMY_TYPES` (grunt, interceptor, tank), `BOSSES` (six bosses, cycled by stage).
- **Audio** — a lazily created Web Audio context (`ensureAudio`), with `beep`, `noiseBurst` and a `sfx` table. Audio starts only after a user gesture.
- **HUD / feedback** — `renderHearts`, `renderBombs`, combo popups, stage banners, screen shake, hit/bomb flashes. Reduced-motion toggles these effects.
- **Scaling** — fixed logical resolution `BASE_W × BASE_H` (480 × 720), scaled to fit the window via `resize()` (max scale 1.6).
- **Input** — keyboard (`keys` map), touch joystick + FIRE/BOMB buttons, and mouse (pointer tracking and hold-to-fire).
- **Game state** — `resetGame()` initializes `player`, `bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`, `boss`, score, lives, bombs, chain/multiplier, stage and kills.
- **Rendering** — `drawPlane`, `drawBossShip` and related canvas draw functions.
- **Update loop** — `spawnEnemy`, collision and damage handling, power-ups (`life`, `bomb`, `weapon`), boss phases with telegraphs, stage progression, and an adaptive `quality` level driven by `trackPerf` (frame-time sampling that lowers effects when frames are slow).

## Controls (for reference when changing input code)

- Keyboard: arrows / WASD to move, Space to fire (held), X or Shift to bomb, P or Escape to pause.
- Touch: left-side joystick, FIRE and BOMB buttons. Touch handlers use `{ passive: false }` and call `preventDefault` where needed.
- Mouse: move to aim the player, hold to fire on the canvas.

## Conventions

- Keep the game as a single-file, dependency-free page unless the user asks for a restructure. Do not add a bundler, framework or package manifest without a clear request.
- Keep all tuning numbers in the existing tables (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `THEMES`) rather than scattering magic numbers through the logic.
- Use the existing `STORAGE_*` constants for any persisted value, and wrap `localStorage` access in a way that tolerates it being unavailable (private modes can throw).
- Preserve the mobile layout: `touch-action: none`, safe-area insets, and the `#touchControls` overlay must keep working at phone width without page scroll.
- Respect the reduced-motion setting when adding new screen shake, flashes or pulses.
- Keep the look consistent with the existing palette (dark blue HUD chrome, gold `#ffd54f` values, green weapon label).

## Verifying changes

There are no automated tests. After editing `index.html`:

1. Open it in a browser (desktop and, if touch code changed, a mobile-sized viewport) and check that the overlay, start button and difficulty buttons work.
2. Play briefly with keyboard and mouse: move, fire, switch weapons (pickup), bomb, pause/resume.
3. Check the browser console for errors.
4. Confirm the game still works with `localStorage` disabled or throwing.

For larger changes, a headless browser check (e.g. Playwright) is a reasonable substitute for manual play.

## Git

- Develop on the branch assigned for the session (currently `claude/gifted-tesla-4hzizg`); do not push to other branches without permission.
- Write clear, descriptive commit messages. Do not open pull requests unless explicitly asked.
