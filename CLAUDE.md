# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo (`ubban/google_4`) is a tiny static project. Its only real content is **Sky Fighter**, a single-page HTML5 canvas vertical shoot-'em-up (Raiden-style) in `index.html`.

- `index.html` — the entire game: inline CSS (`<style>`), markup, and one inline `<script>` wrapped in an IIFE with `'use strict'`. About 1,670 lines.
- `README.md` — a one-line placeholder title. Not documentation of the game.

There is no build step, no package manager, no test suite, and no server-side code.

## Running and checking changes

- Open `index.html` directly in a browser, or serve the folder with any static server (for example `python3 -m http.server`) if a browser setting blocks `file://` features.
- There are no automated tests or linters configured. Verify changes by playing the game in a browser: start a run, check the HUD, pause/resume, use a bomb, and confirm the game-over overlay.
- For UI or gameplay changes, a headless Playwright check is a reasonable way to confirm nothing throws. Use the pre-installed Chromium and do not run `playwright install`.

## Code layout inside `index.html`

The script is organized by section, roughly in this order:

1. **DOM references**: `const xxxEl = document.getElementById(...)`. Every element ID the script uses is declared here. If you add an element ID in markup, add its lookup here too.
2. **Persistence keys**: `STORAGE_*` constants (`skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion`). Read and write these through try/catch, since storage can be unavailable.
3. **Tunables**: `DIFFICULTIES` (spawn rates, enemy speed, shot chance, lives, bombs, boss HP multiplier), `KILLS_PER_STAGE`, `THEMES`, `ENEMY_TYPES`, `BOSSES`. Balance changes usually belong here, not in logic.
4. **Audio**: `ensureAudio()`, `beep()`, `noiseBurst()`, and the `EXPLODE_SFX` map. Audio is created lazily on the first user gesture (`ensureAudio()` is called from input handlers).
5. **HUD and feedback**: `renderHearts`, `renderBombs`, `flashHit`, `flashBomb`, `showComboPop`, `showStageBanner`, `screenShake`, `announce` (screen-reader live region).
6. **Layout and input**: `resize()` scales a `BASE_W × BASE_H` (480 × 720) logical canvas to the viewport. Keyboard state lives in `keys[e.code]`. Touch uses a virtual joystick plus Fire and Bomb buttons.
7. **Game state and entities**: `resetGame()`, `spawnEnemy()`, `spawnBoss()`, `maybeSpawnPowerUp()`, `spawnExplosion()`, scoring with a kill-chain multiplier (`addScore`, `breakChain`), bombs (`useBomb`), and weapons (`fireWeapon`, with three power levels).
8. **Update**: `update(dt)` advances all entities, collisions, and stage progression. `applyPowerUp()` handles pickups.
9. **Bosses**: `BOSS_PATTERNS` is an array of bullet-pattern functions (radial, aimed spread, columns, spiral, cross, homing, double ring). `updateBoss(dt)` and `killBoss()` drive boss phases.
10. **Lifecycle**: `loseLife()`, `endGame()`, `togglePause()`.
11. **Render**: `draw()` plus helpers `drawPlane`, `drawBossShip`, `drawClouds`. Parallax clouds, shake, and the low-health vignette are rendered here.
12. **Main loop**: `loop(now)` uses `requestAnimationFrame`, clamps `dt` to 0.05 s, and calls `update` and `draw` only when running and not paused.

## Controls (for reference when testing)

- Move: arrow keys or WASD. Touch: virtual joystick.
- Fire: hold Space. Touch: Fire button.
- Bomb: `X` or Shift. Touch: Bomb button.
- Pause: `P` or `Esc`.
- Mute, reduced-motion, and difficulty are toggled from the on-screen controls and persisted in `localStorage`.

## Conventions

- Keep the game as a single self-contained HTML file unless the user asks for a split. Do not add a build step or dependencies without a clear reason.
- Match the existing style: 2-space indent, `const`/`let`, arrow functions for callbacks, and section comments where a block is non-obvious.
- Put tunable numbers in the `DIFFICULTIES`, `ENEMY_TYPES`, or `BOSSES` tables rather than hard-coding them in logic.
- Respect the existing accessibility work: keep the `srLive` announcements for important events, and honor the `reducedMotion` setting for new visual effects (see the `body.reduced-motion` rules in the CSS).
- Keep the `touch` and `keyboard` input paths in parity when adding a new action.
- Use `e.preventDefault()` on touch handlers that need it (the buttons use `{ passive: false }`).

## Git workflow

- Development branch for this session: `claude/gifted-tesla-x6mhpq`. Commit and push there with `git push -u origin <branch>`.
- Do not open pull requests unless the user asks for one.
- Do not commit local secrets or environment files; this repo has none today.
