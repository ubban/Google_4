# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo is a single-file browser game, **Sky Fighter**: a Raiden-style vertical shoot-em-up.

- `index.html` is the entire game: markup (`<body>`), CSS (`<style>`, ~lines 10–362), and JavaScript (`<script>`, ~lines 422–1669), with no build step, no dependencies, and no package manager.
- `README.md` is a stub and has no setup instructions.

There is no test suite, linter, or CI configuration in the repo.

## Running the game

No build is needed. Open `index.html` directly in a browser, or serve the directory with any static server, for example:

```bash
python3 -m http.server 8000   # then visit http://localhost:8000/index.html
```

## Code layout inside `index.html`

The script is one IIFE (`(function () { 'use strict'; ... })();`). Everything lives inside it, so nothing is exposed on `window`.

Sections, in file order (search for the `// ---------- ... ----------` comment markers):

- **DOM refs and constants**: cached element lookups, `STORAGE_*` localStorage keys, `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES` (sky palettes per stage), `ENEMY_TYPES`, `BOSSES`.
- **Adaptive performance**: `trackPerf` samples frame times and steps a `quality` level (0–2) up or down to hold ~40–56 fps.
- **Audio**: a Web Audio synth (`ensureAudio`, `beep`, `noiseBurst`). No audio files are loaded.
- **HUD and effects**: hearts, bombs, combo pop, stage banner, screen shake, flash.
- **Viewport and input**: `resize` scales a fixed `BASE_W × BASE_H` (480 × 720) logical canvas to the window. Keyboard state is kept in `keys[e.code]`. Touch uses a joystick plus fire and bomb buttons.
- **Game state**: `resetGame`, `update(dt)`, `draw()`, and `loop(now)`, which drives everything from `requestAnimationFrame` with `dt` capped at 0.05 s.
- **Drawing**: `drawPlane`, `drawBossShip`, `drawClouds`.
- **Spawning and combat**: `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`, `fireWeapon`, `useBomb`, `addScore`/combo chain.
- **Bosses**: `BOSS_PATTERNS` is an array of bullet-pattern functions (radial, aimed spread, columns, spiral, cross, homing, double ring). `updateBoss` and `killBoss` manage phases.
- **Lifecycle**: `loseLife`, `endGame`, `togglePause`.

## Controls

| Action | Keyboard | Mouse | Touch |
| --- | --- | --- | --- |
| Move | Arrow keys or WASD | Move pointer to steer | Joystick (left zone) |
| Fire | Hold Space | Click | Fire button |
| Bomb | X or Shift | Bomb button |
| Pause | P or Escape | — |
| Mute | Mute button | Mute button |

## Persisted state

All persistence goes through `localStorage` under these keys. Wrap any new access in try/catch, because storage can be blocked.

- `skyfighter.bestScore`: high score.
- `skyfighter.muted`: `'1'` when muted.
- `skyfighter.difficulty`: `easy`, `normal`, or `hard`.
- `skyfighter.reducedMotion`: `'1'` or `'0'`. When unset, it falls back to `prefers-reduced-motion`.

## Conventions

- **Keep it in one file** unless the user asks for a split. Match the existing style: 2-space indent, single quotes, `const`/`let`, arrow functions for callbacks, and short section banners.
- **Tune gameplay through the constant tables** (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `THEMES`) rather than hard-coding numbers in logic.
- **Use `dt`-scaled movement** in `update`. Speeds are per-second, so frame rate should not change game speed.
- **Respect `reducedMotion`** when adding screen shake, flashes, or particle-heavy effects. Respect `quality` when adding costly draws.
- **Keep accessibility hooks.** Announce important state changes through `announce()` (the `srLive` live region), and keep `aria-*` attributes on overlays and buttons.
- **Keep touch and keyboard parity.** Any new action needs both a key binding and a touch control.

## Verifying changes

Since there are no automated tests, check changes by playing the game in a browser:

1. Start a game, then check movement, firing, bombs, pause, and game over.
2. Reach a stage boundary (`KILLS_PER_STAGE`) to see the theme change and the stage banner.
3. Reach a boss and check its HP bar and attack patterns.
4. Switch difficulty and confirm the values persist after a reload.
5. Test in a narrow or touch-sized viewport if you touched layout or input.

Use the Playwright install that is already on the machine when you need a headless check. Do not reinstall it.

## Git workflow

- Develop on the branch specified by the session (currently `claude/gifted-tesla-se365x`). Do not push to other branches without permission.
- Commit with clear, descriptive messages. Push with `git push -u origin <branch>`.
- Do not open a pull request unless the user asks for one.
