# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a small, dependency-free static web project. It currently contains a single game:

- **Sky Fighter** (`index.html`): a Raiden-style vertical shoot-'em-up rendered on an HTML5 `<canvas>`. Everything (markup, CSS, JavaScript) lives in one self-contained file, about 1,670 lines.

`README.md` is a placeholder and has no setup instructions.

## Repository layout

```
index.html   Sky Fighter game: <style> (≈lines 10–362), markup (≈363–421), <script> (≈422–1669)
README.md    Placeholder title only
```

There is no build system, package manager, test suite, linter, or CI configuration. Do not add one unless asked.

## Running and testing

- Open `index.html` directly in a browser, or serve the directory with any static server (for example `python3 -m http.server` from the repo root, then visit `http://localhost:8000`).
- There are no automated tests. Verify gameplay changes by loading the page in a browser and playing, or with a headless browser if one is available.
- Check the JavaScript for syntax errors after editing (for example, load the page and watch the console for errors).

## Architecture of `index.html`

The game script is one IIFE (`'use strict'`) with these main parts, in order:

1. **DOM references**: `const` lookups by element id at the top of the script. Keep new element lookups here.
2. **Config constants**: `STORAGE_*` localStorage keys (prefixed `skyfighter.`), `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES`, `ENEMY_TYPES`, `BOSSES`, `BASE_W`/`BASE_H` (480×720 logical canvas size).
3. **Audio**: `ensureAudio()`, `beep()`, `noiseBurst()`, with `EXPLODE_SFX` mapping enemy types to sounds. Audio is created lazily on first user gesture; keep it that way for browser autoplay policies.
4. **UI helpers**: HUD rendering (`renderHearts`, `renderBombs`), feedback effects (`flashHit`, `flashBomb`, `showComboPop`, `showStageBanner`, `screenShake`), and the announcer for screen readers (`announce`).
5. **Input**: keyboard (`keys` map keyed by `e.code`), mouse/pointer via `canvasPointFromEvent`, and touch joystick plus FIRE/BOMB buttons (`joystickStart/Move/End`).
6. **Game state**: `resetGame()` initializes state. Entities are plain objects in arrays (enemies, bullets, power-ups, explosions).
7. **Simulation**: `update(dt)` advances the world each frame. Spawning (`spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`), weapons (`fireWeapon`, `useBomb`), scoring (`addScore`, `breakChain`), and power-ups (`applyPowerUp`) all live here or in helpers it calls.
8. **Bosses**: `BOSS_PATTERNS` is a list of bullet-pattern functions (`fireRadial`, `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`, `fireCross`, `fireHomingSeeded`, `fireDoubleRing`). `updateBoss` drives them and `killBoss` ends a stage.
9. **Rendering**: `draw()` renders everything to the canvas, including parallax clouds (`drawClouds`), planes (`drawPlane`), and boss ships (`drawBossShip`). Screen shake is applied here.
10. **Loop**: `loop(now)` runs on `requestAnimationFrame`, clamps `dt` to 0.05 s, and calls `update` and `draw` only while `running && !paused`.

Game flow is `startBtn` → `resetGame()` → loop → `loseLife()` / `endGame()`. `togglePause()` handles pause, which is also bound to P and Esc.

## Conventions

- **Single file**: keep HTML, CSS, and JS in `index.html` unless the user asks for a split. Do not add external libraries or CDN links without asking.
- **No dependencies**: use plain browser APIs only.
- **Style**: match the existing code. It uses 2-space indentation, single quotes in JS, `const`/`let` (no `var`), arrow functions for callbacks, and short helper functions with descriptive camelCase names.
- **Tuning values**: put gameplay numbers (speeds, spawn rates, damage, score values) in the existing config objects or near the code that uses them, not scattered as unexplained literals.
- **Persistence**: save player settings and best score only through the `skyfighter.*` localStorage keys. New storage access must be wrapped in `try/catch` so the game still works when storage is unavailable (private mode, blocked storage). The existing reads near the top of the script (≈lines 517–522) are not guarded yet; fix that if you touch them.
- **Accessibility**: respect the reduced-motion setting (`STORAGE_REDUCED`) when adding screen shake, flashes, or other motion effects. Keep the `srLive` announcer updated for important state changes.
- **Touch and mouse parity**: any new control must work with the keyboard, mouse, and touch controls. The touch UI is shown only on touch devices (`isTouchDevice`).
- **Performance**: the loop runs every frame. Avoid allocating new objects or large arrays per frame in hot paths, and remove off-screen entities from arrays.

## Controls (for reference when testing)

- Keyboard: arrows or WASD to move, Space to fire, X or Shift for bomb, P or Esc to pause.
- Mouse: move to steer, click to fire.
- Touch: drag left side to fly, FIRE and BOMB buttons on the right.

## Git and workflow

- Default working branch for this repository is `claude/gifted-tesla-iows92`. Commit and push there. Do not push to other branches without explicit permission.
- Keep commits focused. Describe the gameplay or UI change in the commit message body.
- Do not open pull requests unless the user explicitly asks for one.

## Notes for future updates

- If the project grows beyond a single HTML file, update the "Repository layout" and "Architecture" sections to match, and add real build, run, and test commands here.
- When `README.md` gains setup instructions, keep this file consistent with it rather than duplicating them.
