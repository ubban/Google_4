# CLAUDE.md

Guidance for working in this repository.

## What this is

**Sky Fighter** is a Raiden-style vertical shoot-'em-up that runs in the browser. The entire game is one self-contained file, `index.html`: inline `<style>`, the HTML markup, and one inline `<script>` IIFE that holds all game logic and rendering on a single `<canvas id="gameCanvas">`.

`README.md` contains only the project title and a stray timestamp. It has no setup or usage instructions, so treat `index.html` as the source of truth.

## Repository layout

```
index.html   # The whole game: CSS, markup, and JS
README.md    # Title only
```

There is no `package.json`, build step, bundler, test suite, linter, or CI config. Do not add tooling unless asked.

## Running it

Open `index.html` directly in a browser. No server or install step is needed. For a quick check from the shell:

```
xdg-open index.html     # or open / start, depending on the platform
```

## Code map (inside the `<script>` IIFE)

Sections appear roughly in this order. Search for the names rather than relying on line numbers.

| Area | Key names |
| --- | --- |
| DOM refs | `canvas`, `overlay`, and one `const` per element, such as `scoreEl`, `heartsEl`, `bombsEl` |
| Persistence | `STORAGE_BEST`, `STORAGE_MUTE`, `STORAGE_DIFF`, `STORAGE_REDUCED` (all `localStorage` under the `skyfighter.*` prefix) |
| Adaptive quality | `quality`, `trackPerf()`. Drops quality when average frame time is above ~40 fps and raises it when above ~56 fps. |
| Tuning data | `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES` (per-stage sky palette), `ENEMY_TYPES`, `BOSSES` |
| Audio | `ensureAudio()`, `beep()`, `noiseBurst()`, `sfx` and `EXPLODE_SFX`. Web Audio is created lazily on the first user gesture. |
| Input | keyboard (`keys`), touch joystick (`joystickStart/Move/End`), mouse steering on the canvas, `fireBtn`, `bombBtn` |
| Game state | `resetGame()`, `update(dt)`, `draw()`, `loop(now)` |
| Spawning | `spawnEnemy()`, `spawnBoss()`, `maybeSpawnPowerUp()`, `applyPowerUp()` |
| Boss attacks | `BOSS_PATTERNS` and the `fire*` helpers (`fireRadial`, `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`, `fireCross`, `fireHomingSeeded`, `fireDoubleRing`). `updateBoss()` picks patterns. |
| Flow | `loseLife()`, `endGame()`, `killBoss()`, `togglePause()`, `startBtn` click handler |
| Accessibility | `announce()` writes to the `#srLive` live region. `reducedMotion` gates screen shake, flashes, and pulses. |

The game loop is driven by `requestAnimationFrame`. `dt` is clamped to 50 ms, so long stalls do not teleport objects. Keep that clamp in place when changing timing.

## Conventions

- **Single file.** Keep HTML, CSS, and JS in `index.html` unless the user asks for a split.
- **Match the local style.** Use 2-space indentation, single quotes in JS, `const` by default, arrow functions for callbacks, and `// ----------` banner comments to separate sections. Keep comments sparse and explain why, not what.
- **Scope.** Everything sits inside the IIFE. Don't add globals. Use `'use strict'` as the file already does.
- **Tuning lives in data.** Change balance by editing `DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `THEMES`, or the pattern table, not by hardcoding numbers into update logic.
- **Persisted settings** use the `skyfighter.` key prefix. Rename a key only with a migration, because it changes saved player data.
- **Touch and keyboard parity.** Any new control needs a keyboard path, a touch path, and an update to `#controlsHint` in the overlay.
- **Accessibility.** New visual effects that shake or flash must check `reducedMotion`. Status changes that matter to players should go through `announce()`.
- **Mobile.** The canvas scales to a 480×720 logical size via `resize()`. Keep HUD and touch controls usable at phone widths and respect `viewport-fit=cover`.

## Verifying changes

There are no automated tests. After editing:

1. Open `index.html` in a browser (or use the pre-installed Playwright Chromium to load it headlessly).
2. Check the browser console for errors on load and during play.
3. Exercise the paths you touched: start, pause, lose a life, bomb, weapon power-up, stage transition, boss fight, game over, and best-score persistence across reloads.
4. For balance or rendering changes, play at least one stage on each difficulty, and check the touch controls with device emulation.

Report what you actually ran. If you could only check by reading the code, say so.

## Git

- Development branch for this work: `claude/gifted-tesla-qapn0z`. Push with `git push -u origin <branch>`.
- Write clear, present-tense commit messages that describe the gameplay or code change.
- Do not open pull requests unless the user asks for one.
