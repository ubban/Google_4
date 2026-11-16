# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo contains **Sky Fighter**, a Raiden-style vertical shoot-em-up built as a single-file HTML5 canvas game.

- `index.html` — the entire game: markup, CSS, and JavaScript (~1,670 lines).
- `README.md` — minimal placeholder; not a source of project docs.

There is no build step, no bundler, no package manager, no test suite, and no linter configured. Open `index.html` in a browser to run the game.

## Repository layout

```
index.html   # Game: <style> (line ~10), HTML body (~line 360), <script> (~line 422)
README.md    # Placeholder
```

Inside the main `<script>`, everything lives in one IIFE (`(function () { 'use strict'; ... })();`). Nothing is exported or attached to `window`.

## Code map (`index.html` script)

Search for these section anchors when navigating:

| Area | Where to look |
| --- | --- |
| DOM references | Top of IIFE: `document.getElementById(...)` constants |
| Persistence keys | `STORAGE_*` constants (`skyfighter.bestScore`, `.muted`, `.difficulty`, `.reducedMotion`) |
| Adaptive quality | `trackPerf`, `quality` (0 full / 1 medium / 2 low) |
| Tuning tables | `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES`, `ENEMY_TYPES`, `BOSSES` |
| Audio | `ensureAudio`, `beep`, `noiseBurst`, `sfx` |
| HUD/feedback | `renderHearts`, `renderBombs`, `flashHit`, `showComboPop`, `showStageBanner`, `screenShake` |
| Input | Keyboard (`keys`, `keydown`/`keyup`), touch joystick (`joystickStart/Move/End`), fire/bomb buttons, mouse on canvas |
| Canvas sizing | `resize()` — fixed 480×720 logical resolution, scaled to fit the window (max 1.6×) |
| Game state | `resetGame()`, `update(dt)`, `applyPowerUp`, `loseLife`, `endGame`, `togglePause` |
| Weapons & bombs | `fireWeapon`, `useBomb` |
| Enemies & spawning | `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`, `updateBoss` |
| Boss attack patterns | `BOSS_PATTERNS` and the `fire*` helpers (radial, spread, columns, spiral, cross, homing, double ring) |
| Rendering | `drawPlane`, `drawBossShip`, `drawClouds`, `draw()` |
| Main loop | `loop(now)`, started by `requestAnimationFrame(loop)` at the end |

## Game design facts

- **Lives/bombs** come from the difficulty table (`easy`: 4 lives / 3 bombs; `normal`: 3 / 2; `hard`: 3 / 2).
- **Stages**: each stage ends after `KILLS_PER_STAGE` (18) kills, which triggers a boss fight. The boss is `BOSSES[(stage - 1) % BOSSES.length]`, and defeating it advances `stage` (see `killBoss`).
- **Enemies**: `grunt` (stage 1+), `interceptor` (stage 1+, weaves), `tank` (stage 2+, 4 HP).
- **Scoring**: base points per enemy type, multiplied by a kill-chain combo (`addScore`, `breakChain`).
- **Weapons**: `vulcan` and `laser` (`player.weapon`), with a `player.level` power level upgraded via power-ups. A power-up can also toggle the weapon type (see `applyPowerUp`).

## Conventions

- **Keep it a single file** unless the user asks to split it. New CSS goes in the `<style>` block, new JS goes in the existing IIFE, and new markup goes in the body.
- **Stay inside the IIFE.** Don't introduce globals; keep `'use strict'` intact.
- **Use the existing constants.** Tuning values belong in the `DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, or `THEMES` tables rather than hard-coded magic numbers.
- **Use the `STORAGE_*` keys** for any new persisted setting, and prefix new keys with `skyfighter.`.
- **Respect reduced motion.** Effects that shake or flash should check the `reducedMotion` flag, and the `body.reduced-motion` class is available for CSS.
- **Keep accessibility hooks.** Status changes go through `announce()` (the `srLive` live region). Keep `aria-*` attributes and labels on controls intact.
- **Frame-rate independence.** Update logic takes `dt`; keep new movement and timers based on it.
- **Mobile must still work.** Any new control needs a touch path as well as keyboard/mouse, and the layout must work with `touchControls`.

## Working on this repo

- **Verify in a browser.** Since there are no automated tests, load `index.html` in a browser (or serve it with any static server) and play through the affected flow. Check desktop keyboard/mouse and, when touch-related, a touch-emulated viewport.
- **Check the console.** Make sure there are no uncaught errors during play, pause, mute, difficulty changes, and game over.
- **Don't add tooling silently.** Adding a build step, dependencies, or a test framework is a significant change. Ask first.
- **Git.** Work on the branch that the session specifies. Keep commits focused, and use imperative, descriptive commit messages.

## Things to avoid

- Don't reformat or reorganize the whole file in an unrelated change; the file is one large diff surface.
- Don't remove or rename `STORAGE_*` keys, since that would reset players' saved best score, mute, difficulty, and motion settings.
- Don't add external network requests or third-party scripts without being asked. The game is self-contained.
