# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo contains **Sky Fighter**, a single-file, Raiden-style vertical shoot-em-up
built as an HTML5 canvas game. The entire game (markup, CSS, and JavaScript) lives in
`index.html`. `README.md` is a stub and contains only the repo title.

There is no build system, package manager, test suite, linter, or CI configuration.
Do not invent commands for these. If you add one, document it here.

## Repository layout

- `index.html` (~1,700 lines, ~58 KB): the whole game.
  - Lines ~1–362: `<style>`, all CSS. HUD pills, glassmorphic overlay card, touch controls, and the vignette and flash effects.
  - Lines ~363–421: markup. Canvas `#gameCanvas`, HUD elements, overlay, and touch controls.
  - Lines ~422–end: a single `<script>` wrapped in an IIFE with `'use strict'`. No modules, no imports.
- `README.md`: placeholder.
- `CLAUDE.md`: this file.

## Running the game

Open `index.html` directly in a browser. No server or install step is needed.
To test changes, reload the page. Nothing else is required.

## Code architecture (inside the `<script>` IIFE)

The script is organized by section comments (`// ---------- Input ----------` etc.).
The main pieces are:

- **Config tables**: `DIFFICULTIES` (spawn rates, enemy speed, shot chance, lives, bombs,
  boss HP multiplier), `KILLS_PER_STAGE`, `THEMES`, `ENEMY_TYPES`, `BOSSES`, and
  `BOSS_PATTERNS`. Tune gameplay here rather than hard-coding values in update logic.
- **Persistence**: `localStorage` keys prefixed `skyfighter.` (`bestScore`, `muted`,
  `difficulty`, `reducedMotion`). Every access is a bare `localStorage` call, so
  private-mode or disabled-storage errors would surface. Keep that in mind when changing it.
- **Audio**: a Web Audio API layer (`ensureAudio`, `beep`, `noiseBurst`). Audio starts
  only after a user gesture, which `ensureAudio()` handles.
- **Input**: keyboard (`keys` map, `keydown`/`keyup` on `window`), mouse and pointer
  steering via `canvasPointFromEvent`, and touch joystick plus FIRE and BOMB buttons,
  enabled only when `isTouchDevice` is true.
- **Scaling**: the canvas is a fixed logical `BASE_W` x `BASE_H` (480 x 720). `resize()`
  scales it for display. Game coordinates always use the base size.
- **Loop**: `requestAnimationFrame(loop)` calls `update(dt)` then `draw()`. `dt` is clamped
  to 0.05 s so tab switches don't cause large jumps. Updates are skipped while paused or
  not running.
- **Game state**: `resetGame()` initializes state. `update()` handles the simulation. It
  covers enemies, bullets, power-ups, combo and score, and stage progression. `endGame()`
  and `loseLife()` handle death and game over. `togglePause()` handles pause.
- **Weapons and bombs**: `fireWeapon()` with 3 power levels (Vulcan and Laser), `useBomb()`
  for screen-clearing bombs that also damage bosses, and `applyPowerUp()` for drops.
- **Bosses**: `spawnBoss()`, `updateBoss()`, and `killBoss()`. Attacks are functions
  such as `fireRadial`, `fireAimedSpread`, and `fireSpiralBurst`, selected from `BOSS_PATTERNS`.
- **Drawing**: `draw()` renders everything, with helpers `drawPlane`, `drawBossShip`,
  `drawClouds`, and `spawnExplosion`. Screen shake and hit flash are also handled here.
- **Accessibility**: a reduced-motion setting toggles animation effects. There is an
  ARIA live region (`srLive`) fed through `announce()`. Keep these in sync when you add
  new feedback.

## Conventions

- Keep everything in `index.html` unless the user asks for a split. Don't add a build
  step or dependencies without a clear reason.
- Use `const` and `let`. The script is in strict mode, so don't rely on implicit globals.
- Match the existing style: two-space indentation, single quotes in JS, short section
  comments, and small named helper functions.
- Put tunable numbers in the config tables, not inline in logic.
- When you add UI text, update both the markup and any relevant `announce()` calls.
- Keep game-facing text consistent with the controls hint on line ~417 (the `controlsHint`
  element). Update it if controls change.

## Testing and verification

There is no automated test suite. Verify changes manually:

1. Open `index.html` in a browser and check the console for errors.
2. Play through start, pause (P or Esc), bombs (X or Shift), a boss, and game over.
3. Check that the best score, mute, difficulty, and reduced-motion settings persist across reloads.
4. For touch changes, test with device emulation or a touch device.

If you use Playwright or a headless browser, do not install it yourself if a browser is
already available in the environment. Use the existing installation.

## Git workflow

- Develop on the branch designated for the session (currently `claude/gifted-tesla-ji611w`).
- Commit messages describe the change in plain terms. The existing history uses a short
  subject line followed by bulleted detail.
- Don't open pull requests unless asked.
