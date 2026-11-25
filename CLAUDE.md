# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo is **Sky Fighter**, a single-file browser game: a Raiden-style vertical shoot-'em-up. There is no build system, package manager, test suite, or server-side code.

- `index.html` (~1,670 lines) contains all markup, CSS, and JavaScript inline. Its `<title>` is "Sky Fighter".
- `README.md` contains only the project title.

To run the game, open `index.html` in a browser, or serve the directory with any static file server.

## Repository layout

```
index.html   # entire game: <style> block, #container/#hud/#overlay DOM, one <script> block
README.md    # title only
CLAUDE.md    # this file
```

Inside `index.html`, the rough order is:

1. **CSS** (top of file): dark color scheme, full-viewport `#container`, HUD (`#hud`, `.stat` pills), touch controls, overlay, and keyframes such as `heartBeat`.
2. **DOM** (~line 362): `#gameCanvas`, `#hud` stats (`#score`, `#best`, `#mult`, `#weaponLvl`, `#hearts`, `#bombs`), `#bossBarWrap`, `#topBar` buttons (`#muteBtn`, `#motionBtn`, `#pauseBtn`), `#touchControls` (`#moveZone`, `#fireBtn`, `#bombBtn`), and `#overlay`.
3. **Script** (~line 460 onward), with the game constants and state at the top:
   - `STORAGE_*` keys for `localStorage` (`skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion`).
   - `DIFFICULTIES` (easy / normal / hard): spawn rates, enemy speed and fire chance, starting lives and bombs, and boss HP multiplier.
   - `KILLS_PER_STAGE` (18), `THEMES` (six sky palettes that cycle per stage), `ENEMY_TYPES` (grunt, interceptor, tank), and `BOSSES` (six bosses with names and HP multipliers).
   - Audio (`ensureAudio`, `beep`, `noiseBurst`), UI effects (hearts, bombs, flash, combo pop, stage banner, screen shake), and input handling (keyboard, touch joystick, mouse).
   - Game logic: `resetGame`, `update(dt)`, `spawnEnemy`, `spawnBoss`, `fireWeapon`, `useBomb`, `applyPowerUp`, and boss bullet patterns (`fireRadial`, `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`, `fireCross`).
   - Rendering: `drawPlane`, `drawBossShip`, and the main draw loop driven by `requestAnimationFrame`.

## Controls (for reference when testing)

- **Keyboard:** arrow keys or WASD to move (via `keys[e.code]`), Space to fire, X / Shift to bomb, P / Escape to pause.
- **Touch:** left-side joystick, FIRE and BOMB buttons on the right.
- **Mouse:** move the pointer over the canvas to steer, and hold the mouse button to fire.

## Conventions

- Keep the game in a single `index.html` unless the user asks for a split. Do not add a build step, bundler, or framework without asking.
- Keep tunable values in the existing constant objects (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `THEMES`) rather than hard-coding them inside functions.
- Persist settings only through the `STORAGE_*` keys, and wrap storage access so the game still runs if `localStorage` is unavailable.
- The game uses a fixed logical canvas of `BASE_W = 480` × `BASE_H = 720`, scaled by `resize()`. Keep new positions and sizes in those logical units.
- Preserve the mobile layout: the viewport meta disables zoom, `touch-action: none` is set, and touch controls must stay usable at phone width.
- Respect the reduced-motion toggle (`reducedMotion`) for any new screen shake, flashing, or pulsing effect.
- Keep the code plain ES2015+ JavaScript with no external dependencies. Match the surrounding indentation (2 spaces) and style.

## Verification

There are no automated tests or linters. After changing game logic or markup:

1. Open `index.html` in a browser and check the browser console for errors.
2. Play through at least one stage on each difficulty you touched, and confirm that the HUD, bosses, and stage transitions still work.
3. For layout changes, check a narrow (phone-width) viewport as well as a desktop one.

For a headless check, a Playwright script against `file://…/index.html` is sufficient.

## Git

- Development branch: `claude/gifted-tesla-e3spiz`. Push there with `git push -u origin <branch>`.
- Do not open a pull request unless the user asks for one.
- Commit messages should be short and describe what changed in the game (for example, "Add Volcanic Ridge stage theme").
