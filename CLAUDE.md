# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Google_4 is a small static repo containing **Sky Fighter**, a Raiden-style vertical shoot-'em-up
written as a single-file HTML5 canvas game.

- `index.html` — the entire game: inline CSS, markup, and one inline `<script>` IIFE (~1,670 lines).
- `README.md` — minimal placeholder.

There is no build step, no bundler, no `package.json`, no test suite, and no linter configured.
No external dependencies are loaded; everything (graphics, sound) is drawn or synthesized in code.

## Running the game

Open `index.html` directly in a browser, or serve the directory with any static server, e.g.:

```
python3 -m http.server 8000
```

Then visit `http://localhost:8000/index.html`. There are no install or build commands.

## Code layout inside `index.html`

The script is wrapped in `(function () { 'use strict'; ... })()`. Roughly, top to bottom:

1. **DOM references** — `getElementById` lookups for HUD, overlay, touch controls, and boss bar.
2. **Storage keys** — `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion` (all in `localStorage`).
3. **Config tables** — `DIFFICULTIES` (easy/normal/hard), `THEMES` (per-stage sky palettes), `ENEMY_TYPES`, `BOSSES`, `KILLS_PER_STAGE`.
4. **Audio** — `ensureAudio()`, `beep()`, `noiseBurst()`, and a named `sfx` object. Audio is created lazily on first user gesture.
5. **HUD / feedback** — `renderHearts`, `renderBombs`, `showComboPop`, `showStageBanner`, `screenShake`, `flashHit`.
6. **Input** — keyboard (`keys[e.code]`), touch joystick + FIRE/BOMB buttons, and mouse steering on the canvas.
7. **Game state** — `resetGame()` initialises run state; `update(dt)` is the per-frame simulation.
8. **Entities & spawning** — `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`, `spawnExplosion`.
9. **Weapons & bosses** — `fireWeapon`, `applyPowerUp`, and boss attack patterns in `BOSS_PATTERNS` (`fireRadial`, `fireAimedSpread`, `fireCross`, `fireHomingSeeded`, etc.) driven by `updateBoss`.
10. **Rendering** — `drawPlane`, `drawBossShip`, and the draw loop.
11. **Main loop & UI wiring** — pause/resume, game over, best-score persistence, responsive `resize()`, and the adaptive quality tracker `trackPerf()`.

## Conventions and gotchas

- **Keep it a single file** unless the user asks for a split. Styles stay in the `<style>` block and logic in the single `<script>` block.
- **Plain JavaScript only** — ES2015+ features and Canvas 2D APIs already in use. Do not introduce a framework or build tool without asking.
- **Preserve the IIFE and `'use strict'`.** Game state is `let` variables inside the closure; do not leak globals.
- **Tuning lives in the config tables.** Prefer changing `DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `THEMES`, or `KILLS_PER_STAGE` over hard-coding numbers in logic.
- **Fixed logical canvas size** is `BASE_W = 480`, `BASE_H = 720`; `resize()` scales it to the viewport. Coordinates in game logic use these logical units.
- **Frame timing** uses a `dt` value passed to `update(dt)`. Keep movement and timers `dt`-based, not per-frame-count based, so behaviour holds across refresh rates.
- **Adaptive quality** (`quality` 0–2, via `trackPerf`) reduces effects when frame time is high. New visual effects should respect it.
- **Accessibility:** honour `reducedMotion` (`body.reduced-motion` class turns off several CSS animations, and `screenShake()` dampens shake magnitude and duration), keep the `#srLive` announcements for important state changes, and keep the mute/motion/pause controls working.
- **Mobile:** the page uses `touch-action: none` and `user-select: none`. Any new control must work with touch and must call `preventDefault()` where needed (see the existing FIRE/BOMB handlers, which use `{ passive: false }`).
- **Audio** only starts after a user gesture (browser autoplay policy). Call `ensureAudio()` from input handlers before playing sounds.
- **Persisted values** are read with `localStorage.getItem` at startup. Guard any new storage access, since it may be unavailable in some browsers or private modes.

## Controls (for reference when changing input code)

| Action | Keyboard | Mouse | Touch |
| --- | --- | --- | --- |
| Move | Arrows / WASD | Move cursor over canvas | Drag left zone |
| Fire | Space (hold) | Click / hold | FIRE button |
| Bomb | X or Shift | — | BOMB button |
| Pause | P or Esc | Pause button | Pause button |

## Making changes

- Read the relevant section of `index.html` before editing; the file is large, so search for the function or table you need.
- There are no automated tests. Verify changes by loading the page in a browser and playing the affected path (start, fire, bomb, boss, pause, game over, and the mobile layout if the change touches layout or input).
- Keep commits focused. Use clear messages that describe the gameplay or UI effect of the change.
