# CLAUDE.md

Guidance for Claude Code when working in this repository.

## What this repo is

`Google_4` is a small static web project containing **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs in the browser on a `<canvas>`.

| File | Purpose |
| --- | --- |
| `index.html` | The entire game: markup, CSS (`<style>`, ~lines 10–362), and JavaScript (`<script>`, ~lines 422–1669). |
| `README.md` | Minimal project readme. |

There is no build step, package manager, bundler, test suite, or linter. Do not add one unless asked.

## Running the game

Open `index.html` directly in a browser, or serve the directory with any static server, for example:

```bash
python3 -m http.server 8000   # then visit http://localhost:8000
```

Verify changes in a real browser when you can (see the `run` skill). Tests and linting do not exist here, so reading the code and exercising the game are the only checks.

## Code architecture (inside `index.html`)

The script is one IIFE with `'use strict'`. Everything is private to that closure; nothing is exposed on `window`.

Major sections, in file order:

1. **DOM references**: `const x = document.getElementById(...)` block at the top of the IIFE.
2. **Storage keys**: `STORAGE_BEST`, `STORAGE_MUTE`, `STORAGE_DIFF`, `STORAGE_REDUCED`, all prefixed `skyfighter.`.
3. **Config tables**: `DIFFICULTIES` (spawn rates, enemy speed, lives, bombs, boss HP multiplier), `KILLS_PER_STAGE`, `THEMES` (per-stage sky palettes), `ENEMY_TYPES`, `BOSSES`.
4. **Audio**: `ensureAudio()`, `beep()`, `noiseBurst()`, and `EXPLODE_SFX`. Audio is synthesized with the Web Audio API; there are no audio files.
5. **HUD and effects**: `renderHearts`, `renderBombs`, `flashHit`, `showComboPop`, `showStageBanner`, `screenShake`.
6. **Layout and scaling**: `BASE_W = 480`, `BASE_H = 720`, `resize()`. The canvas is drawn at logical 480×720 and scaled to fit.
7. **Input**: keyboard handlers on `window`, a virtual joystick and fire/bomb buttons for touch, and `canvasPointFromEvent`.
8. **Game state and loop**: `resetGame()`, `update(dt)`, `spawnEnemy`, `spawnBoss`, `applyPowerUp`, `useBomb`, `fireWeapon`, `loseLife`, `endGame`, `togglePause`.
9. **Boss patterns**: `BOSS_PATTERNS` is an array of bullet-pattern functions (`fireRadial`, `fireCross`, `fireSpiralBurst`, and so on). `updateBoss` and `killBoss` drive them.
10. **Rendering**: `draw()`, `drawPlane`, `drawBossShip`, `drawClouds`. Drawing is done on the 2D context each frame.
11. **Main loop**: `loop(now)` uses `requestAnimationFrame`, clamps `dt` to 0.05 s, calls `trackPerf`, then `update` and `draw` while running and not paused.

### Controls (keep docs and code in sync)

- Move: arrow keys or `WASD`.
- Fire: hold `Space`, or the touch fire button.
- Bomb: `X` or either `Shift`, or the touch bomb button.
- Pause: `P` or `Escape`.
- Mouse: moving over the canvas moves the ship toward the cursor; pressing the button on the canvas fires (`mousedown` sets `firing`).
- Touch: the virtual joystick in the bottom zone moves the ship; the on-screen fire and bomb buttons work as above.

### Persistence

Player settings and the best score live in `localStorage`. Every access must be safe to fail. Private browsing and blocked storage can throw, so wrap reads and writes in `try/catch` and fall back to in-memory defaults if that happens. Existing code reads `localStorage` without a guard; when you touch those lines, add the guard.

### Accessibility

- A `prefers-reduced-motion` media query sets the default for the reduced-motion setting, which the user can override and store under `STORAGE_REDUCED`. Honor it when adding screen shake, flashes, or other motion effects.
- `announce()` writes to the `srLive` ARIA live region. Use it for important state changes such as stage starts, boss arrivals, and game over.

## Conventions

- **Single file.** Keep the game in `index.html` unless the user asks for a split. Do not introduce external dependencies, CDN scripts, or frameworks.
- **No build artifacts.** Do not commit generated or minified output.
- **Match the existing style.** Two-space indentation, single quotes in JS, `const` by default, `let` only when reassigned, arrow functions for callbacks, and short helper functions with descriptive names.
- **Preserve the IIFE.** New top-level code goes inside the existing `(function () { ... })();` block so it stays private.
- **Frame-rate independence.** Movement and timers should use `dt` (seconds), as `update(dt)` does, not frame counts.
- **Balance changes** belong in the config tables (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `BOSS_PATTERNS`) rather than being hard-coded in logic.
- **Mobile first.** The layout must work on phones: keep the 16px gutter, avoid horizontal page scroll, and keep the touch controls usable. The viewport meta disables user scaling; do not remove that without asking.

## Git and branches

- Development branch for this work: `claude/gifted-tesla-q1m0ct`. Commit and push there. Do not push to other branches without explicit permission.
- Push with `git push -u origin <branch>`. Retry network failures with backoff.
- Do not open a pull request unless the user explicitly asks for one.
- Do not force-push. If a push is rejected, fetch and merge the remote branch first.

## Working in this repo

- Read the relevant section of `index.html` before editing; the file is large (~1,670 lines) and most changes touch only one or two functions.
- After a gameplay change, play the game briefly in a browser to confirm it still runs. Check the console for errors.
- Keep this file current. When you add a major section, a new storage key, a control, or a new config table, update the relevant part above.
