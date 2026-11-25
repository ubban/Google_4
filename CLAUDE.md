# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Google_4 is a small static web project. Its only game is **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs entirely in the browser.

- `index.html` — the whole game: inline CSS, markup, and one inline `<script>` (an IIFE). No build step, no bundler, no dependencies.
- `README.md` — a one-line title. It is not a real user doc.

There are no tests, no package manifest, and no CI configuration.

## Running and verifying

- Open `index.html` directly in a browser, or serve the directory with any static server (for example `python3 -m http.server`).
- There is no build, lint, or test command. Verify changes by loading the page and playing it. Check desktop keyboard and mouse controls, and touch controls at phone width.

## Code layout inside `index.html`

The inline script is organized roughly in this order:

1. **DOM refs and constants** — element lookups, `STORAGE_*` keys (`skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion`), `KILLS_PER_STAGE`, `DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `THEMES`.
2. **Audio** — `ensureAudio`, `beep`, `noiseBurst`, and the `sfx` helpers. Audio is generated with the Web Audio API, not sound files.
3. **HUD and effects** — hearts, bombs, combo pops, stage banners, screen shake, and flash.
4. **Input** — `keydown`/`keyup` handlers, joystick touch controls, and mouse handling on the canvas.
5. **Game state** — `resetGame`, `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`, `addScore`, `useBomb`, `fireWeapon`.
6. **Update and boss patterns** — `update(dt)`, `applyPowerUp`, the `fire*` pattern functions for bosses, `updateBoss`, `killBoss`, `loseLife`, `endGame`, `togglePause`.
7. **Rendering** — `draw()` and the `draw*` helpers.
8. **Loop** — `loop(now)` drives everything through `requestAnimationFrame`.

Game state lives in `let` variables inside the IIFE. Keep new state there so the file stays self-contained.

## Conventions

- **Keep it one file.** Put new CSS, markup, and JS in `index.html` unless the user asks for a split. Do not add a build step or a framework without asking.
- **Keep the game-loop shape.** Game logic goes in `update(dt)` and drawing goes in `draw()`. Do not mix them.
- **Persist only through the existing `STORAGE_*` keys** and wrap `localStorage` access so the game still runs when storage is blocked or unavailable.
- **Respect the reduced-motion setting.** Screen shake, flashes, and pulses should check `reducedMotion` before running.
- **Keep accessibility hooks.** Changes that affect game state should keep the `srLive` announcements (`announce`) in step with what the player sees.
- **Support both input methods.** Any new control needs a keyboard path and a touch path (`touchControls`). Mouse steering is also supported.
- **Mobile layout.** The page is `overflow: hidden` with `touch-action: none`. Keep the HUD and touch controls usable at phone width.

## Gameplay tuning

Most balance values sit in the constants near the top of the script: `DIFFICULTIES`, `ENEMY_TYPES` (including `minStage`), `BOSSES`, `THEMES`, and `KILLS_PER_STAGE`. Change these rather than hard-coding values in the update logic.

## Git

- Development branch: `claude/gifted-tesla-9x2db1`. Push there with `git push -u origin <branch>`.
- Do not open a pull request unless the user asks for one.
- Do not commit secrets.
