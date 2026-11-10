# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a single-file browser game, **Sky Fighter**: a Raiden-style vertical shoot-'em-up. Switch weapons, drop smart bombs, and take down stage bosses.

There is no build step, no package manager, and no test suite. The whole game lives in one file:

| File | Purpose |
| --- | --- |
| `index.html` | Complete game: inline CSS (`<style>`), HTML overlay/HUD markup, and one IIFE `<script>` containing all game logic (~1,670 lines) |
| `README.md` | One-line project title |
| `CLAUDE.md` | This file |

To play, open `index.html` in a browser (or serve the directory with any static server). Nothing needs to be installed.

## Repository layout and conventions

- Keep the game self-contained in `index.html`. Do not add a bundler, framework, or external script/CSS dependency unless the user asks for it.
- All JavaScript is wrapped in `(function () { 'use strict'; ... })();`. Do not leak globals; new code should live inside that IIFE.
- Use the existing style: `const` by default, `let` for mutable state, 2-space indentation, single quotes in JS.
- The canvas has a fixed logical size of `BASE_W = 480` x `BASE_H = 720`; `resize()` scales it to the viewport (capped at 1.6x). Coordinates in game code are in these logical units.

## Architecture of `index.html`

Sections of the inline `<script>`, roughly in file order:

1. **DOM references and storage keys.** `STORAGE_*` constants hold the `localStorage` keys: `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion`.
2. **Tuning tables.**
   - `DIFFICULTIES` (`easy` / `normal` / `hard`): spawn timing, enemy speed, shoot chance, starting lives and bombs, boss HP multiplier.
   - `KILLS_PER_STAGE` (18): kills needed to trigger a boss.
   - `THEMES`: six sky palettes, cycled per stage.
   - `ENEMY_TYPES`: `grunt`, `interceptor` (weaving), `tank` (4 HP, unlocks at stage 2).
   - `BOSSES`: six boss designs, selected by `(stage - 1) % BOSSES.length`.
3. **Audio.** Web Audio synthesis only (`beep`, `noiseBurst`, the `sfx` table). No audio files. `ensureAudio()` must run from a user gesture.
4. **Input.** Keyboard (`keys`), touch joystick (`moveZone`), fire/bomb buttons, and mouse steering on the canvas.
5. **State.** Module-level `let` variables: `player`, `bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`, `boss`, plus `score`, `lives`, `bombCount`, `stage`, `chain`/`multiplier`, etc.
6. **Game flow.** `resetGame()` initializes a run; `update(dt)` advances simulation; `updateBoss(dt)`; `applyPowerUp(type)`; `useBomb()`; `fireWeapon()`; `endGame()`; `togglePause()`.
7. **Rendering.** `draw()` plus helpers `drawPlane`, `drawBossShip`, `drawClouds`. HUD updates go through `renderHearts`, `renderBombs`, and the DOM elements cached at the top.
8. **Main loop.** `loop(now)` runs on `requestAnimationFrame`. A simple perf tracker (`trackPerf`) lowers a `quality` level (0 full, 1 medium, 2 low) when frame times are poor.

Power-up types are `life`, `bomb`, and `weapon`, spawned by `maybeSpawnPowerUp`.

## Accessibility and user settings

These are part of the product, so keep them working when you change related code:

- **Reduced motion**: stored preference, defaults to the `prefers-reduced-motion` media query. Toggled by `motionBtn`.
- **Mute**: toggled by `muteBtn`; persisted.
- **Difficulty**: selected from the `.diffBtn` buttons; persisted.
- **Screen readers**: announcements go through `announce()` into the `#srLive` live region.
- **Pause**: `P` or `Esc` on desktop; the pause button on mobile.

Storage access is currently unguarded (`localStorage.getItem` / `setItem` directly). If you touch those calls, consider wrapping them in `try/catch` so the game still runs in private browsing or with blocked storage.

## Controls (for reference when testing)

- Desktop: mouse to steer, click or Space to fire, Arrows/WASD to move, X or Shift for bomb, P/Esc to pause.
- Touch: drag left side to fly, tap FIRE, tap BOMB. The page disables scrolling and zoom (`touch-action: none`, `user-scalable=no`).

## Working in this repo

- **Verify in a browser.** There are no automated tests. For gameplay or layout changes, open `index.html` in a headless browser (Playwright is available in the cloud environment) and check the start screen, a short play session, and the game-over overlay. Check a narrow mobile viewport as well as desktop.
- **Keep changes small and local.** The file is monolithic, so prefer targeted edits over rewriting large sections, and do not reformat unrelated code.
- **Balance numbers live in the tables** near the top of the script (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `KILLS_PER_STAGE`). Tune there rather than hard-coding values in logic.
- **Git.** Work on the branch you were assigned. Commit with clear messages and push with `git push -u origin <branch>`. Do not open a pull request unless asked.

## Known gaps

- No tests, linting, or CI configuration exist.
- `README.md` is only a title; player-facing documentation lives in the on-page overlay text.
- `index.html` is large enough that a future split into separate CSS/JS files may be worth considering, but that is not required today.
