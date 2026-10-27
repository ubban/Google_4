# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo is **Sky Fighter**, a Raiden-style vertical shoot-em-up that runs in the browser. The entire game is one self-contained file, `index.html`, with inline CSS and JavaScript and an HTML5 `<canvas>`. `README.md` is a placeholder and does not describe the game.

There is no build step, no package manager, no bundler, no framework, and no test suite. Do not add any of these unless the user asks.

## Running the game

Open `index.html` directly in a browser, or serve the directory with any static server, for example:

```
python3 -m http.server 8000
```

Then visit `http://localhost:8000/`.

## File layout

| Path | Contents |
| --- | --- |
| `index.html` | The whole game: markup (~lines 1–420), CSS (~lines 10–360), and one IIFE holding all game code in a single `<script>` block (~lines 422–1667) |
| `README.md` | Placeholder |
| `CLAUDE.md` | This file |

Line numbers are approximate and will drift as the file changes.

## Code structure (inside the script IIFE)

The script is wrapped in `(function () { 'use strict'; ... })()`. Everything is private to that closure. Keep new code inside it.

Rough order of sections:

1. **DOM references**: `const xxxEl = document.getElementById(...)` at the top.
2. **Storage keys**: `STORAGE_BEST`, `STORAGE_MUTE`, `STORAGE_DIFF`, `STORAGE_REDUCED`. All are `skyfighter.*` names in `localStorage`.
3. **Adaptive quality**: `trackPerf(dt)` samples the last 45 frame times and steps `quality` between 0 (full), 1 (medium), and 2 (low).
4. **Tuning data**:
   - `DIFFICULTIES`: `easy`, `normal`, `hard`, each with `spawnBase`, `spawnMin`, `enemySpeed`, `enemyShootChance`, `lives`, `bombs`, and `bossHpMul`.
   - `KILLS_PER_STAGE = 18`.
   - `THEMES`: six sky palettes that cycle with the stage.
   - `ENEMY_TYPES`: `grunt`, `interceptor` (weaving), and `tank` (4 HP, from stage 2).
   - `BOSSES`: six bosses, each with a name, colors, and an `hpMul`.
   - `BOSS_PATTERNS` (~line 1372): bullet patterns such as radial, aimed spread, spiral, cross, homing, and double ring.
5. **Audio**: a lazily created `AudioContext` (`ensureAudio()`), with `beep()` and `noiseBurst()` as primitives and an `sfx` object built on them. Audio is created on the first user gesture.
6. **HUD helpers**: `renderHearts`, `renderBombs`, `flashHit`, `showComboPop`, `showStageBanner`, `screenShake`.
7. **Input and layout**: `resize()` scales a fixed `480 × 720` logical canvas to fit the window (max scale 1.6). Keyboard, mouse, and touch handlers are registered here.
8. **Game state and simulation**: `resetGame()`, `update(dt)`, spawning functions, `fireWeapon()`, `useBomb()`, `applyPowerUp()`, `updateBoss(dt)`, `killBoss()`, `loseLife()`, `endGame()`, `togglePause()`.
9. **Rendering**: `draw()` and its helpers (`drawPlane`, `drawBossShip`, `drawClouds`).
10. **Main loop**: `loop(now)` is driven by `requestAnimationFrame`. It caps `dt` at 0.05 s, calls `update(dt)` and `draw()` only while `running && !paused`, and then re-schedules itself.

## Controls

| Action | Keyboard | Mouse / touch |
| --- | --- | --- |
| Move | Arrow keys or WASD | Mouse hover over canvas steers the plane. Touch uses a joystick in the left zone. |
| Fire | Hold Space | Hold the fire button, or hold the mouse button |
| Bomb | X or Shift | Bomb button (tap or click) |
| Pause | P or Escape | Pause button |

Space's default is prevented so the page does not scroll.

## Game rules worth knowing

- **Lives and bombs** come from the chosen difficulty. Losing a life gives 1.5 s of invulnerability and breaks the combo chain.
- **Combo chain**: each kill calls `addScore()`, which increments `chain`, resets `chainTimer` to 2.2 s, and sets `multiplier = 1 + min(9, floor(chain / 5))`. The chain also breaks when the timer runs out.
- **Power-ups** (`life`, `bomb`, `weapon`) are dropped by `maybeSpawnPowerUp()` and applied in `applyPowerUp()`.
- **Stages**: after `KILLS_PER_STAGE` (18) stage kills, a boss spawns. Defeating it calls `killBoss()`, which increments `stage`. The boss is `BOSSES[(stage - 1) % 6]`, with HP `(70 + stage × 35) × difficulty.bossHpMul × boss.hpMul`. The sky palette is `THEMES[(stage - 1) % 6]`.
- **Best score** is saved to `localStorage` under `skyfighter.bestScore` when a game ends.

## Persisted settings

All persistence uses `localStorage`. Each read and write is a plain call with no `try/catch`.

| Key | Values |
| --- | --- |
| `skyfighter.bestScore` | Number |
| `skyfighter.muted` | `'1'` or `'0'` |
| `skyfighter.difficulty` | `easy`, `normal`, or `hard` |
| `skyfighter.reducedMotion` | `'1'` or `'0'`. If unset, it follows `prefers-reduced-motion`. |

If you change the storage layer, keep these keys stable so players keep their saved settings.

## Conventions

- **Single file.** Keep HTML, CSS, and JS in `index.html` unless the user asks for a split. Do not introduce a module system.
- **No dependencies.** Use plain browser APIs only. Do not add CDN scripts.
- **Match the local style.** The code uses 2-space indentation, single quotes in JS, `const`/`let` (no `var`), arrow functions for callbacks, and short section banners in the form `// ---------- Name ----------`. Follow these patterns.
- **Keep tuning in data.** Put balance numbers in the existing tables (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`) rather than as magic numbers inside functions.
- **Accessibility.** Announcements go through `announce()`, which writes to the `#srLive` live region. Respect the reduced-motion setting when adding screen-shake or flash effects.
- **Keep the logical resolution fixed** at 480 × 720. Scaling is handled by CSS through `resize()`.

## Verifying changes

There are no automated tests or linters. To verify a change:

1. Open `index.html` in a browser, or serve it locally as described above.
2. Check the browser console for errors on load and during play.
3. Play through the affected path: start a game, pause and resume, lose a life, use a bomb, and reach a boss if the change touches bosses.
4. For HUD or layout changes, check both a desktop window and a narrow, phone-sized viewport, including the touch controls.
5. For audio changes, confirm that mute still works and that audio starts only after a user gesture.

## Git

- Development happens on feature branches named `claude/...`. Do not push to `main` without explicit permission.
- Commit messages should be short and describe the gameplay or code change.
