# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo is a small, dependency-free browser game called **Sky Fighter**, a Raiden-style vertical shoot-'em-up. The entire game lives in one file, `index.html` (~1,670 lines): inline `<style>`, markup, and one inline `<script>` wrapped in an IIFE. `README.md` is a placeholder and has no build or run instructions.

There is no build step, package manager, test suite, linter, or CI configuration.

## Repository layout

```
.
├── index.html   # The whole game: CSS, HTML, and JS (canvas 2D rendering, Web Audio SFX)
├── README.md    # Placeholder
└── CLAUDE.md    # This file
```

## Running the game

Open `index.html` directly in a browser, or serve the directory with any static server, for example:

```bash
python3 -m http.server 8000   # then visit http://localhost:8000
```

Nothing needs to be installed.

## Architecture of `index.html`

Everything below is inside the single IIFE in the inline `<script>`.

- **Canvas**: `#gameCanvas` drawn with the 2D context. The logical play field is `BASE_W = 480` × `BASE_H = 720`, scaled by `resize()` to fit the viewport.
- **Main loop**: `loop(now)` runs on `requestAnimationFrame`. It clamps `dt` to 0.05 s, calls `trackPerf(dt)`, then `update(dt)` and `draw()` while the game is running and not paused.
- **Adaptive quality**: `trackPerf` keeps a 45-frame rolling average and steps `quality` (0 full, 1 medium, 2 low) down when frames are slow and back up when they are fast.
- **Difficulty**: `DIFFICULTIES` (`easy` / `normal` / `hard`) sets spawn rates, enemy speed, enemy fire chance, lives, bombs, and boss HP multiplier. `setDifficulty()` persists the choice.
- **Stages**: `KILLS_PER_STAGE = 18`. Each stage cycles through `THEMES` (sky palettes and cloud/sun colors).
- **Enemies and bosses**: `ENEMY_TYPES` (grunt, interceptor, tank; `minStage` gates when they appear) and `BOSSES` (six bosses, each with an `hpMul`). Boss attacks are the `fire*` pattern helpers plus `BOSS_PATTERNS`, driven by `updateBoss()`.
- **Game state**: `resetGame()`, `update()`, `loseLife()`, `endGame()`, `togglePause()`. The best score is saved to `localStorage`.
- **Audio**: Web Audio helpers (`ensureAudio`, `beep`, `noiseBurst`) with no audio files. Audio is created lazily on the first user gesture. Mute state is stored.
- **Input**: keyboard (`keys` map on `keydown`/`keyup`), mouse (steer and click-to-fire on the canvas), and touch (joystick on the left, FIRE and BOMB buttons on the right, via `touchControls`).
- **Accessibility**: `announce()` writes to the `#srLive` live region. A reduced-motion toggle is stored and defaults to `prefers-reduced-motion`. When it is on, `screenShake()` cuts magnitude to 15% and caps duration at 0.1 s, and `flashHit()` / `flashBomb()` return without flashing.

### Persisted keys (`localStorage`)

| Key | Meaning |
| --- | --- |
| `skyfighter.bestScore` | Best score (number) |
| `skyfighter.muted` | `'1'` if muted |
| `skyfighter.difficulty` | `easy`, `normal`, or `hard` |
| `skyfighter.reducedMotion` | `'1'` or `'0'`; unset means follow the OS preference |

Keep these key names stable. Changing one silently resets that player's saved data.

### Controls (from the in-game hint)

- Mouse: move to steer, click to fire
- Arrows or WASD: move; Space: fire
- X or Shift: bomb
- P or Esc: pause
- Mobile: drag left to fly, tap FIRE, tap BOMB

## Conventions

- Keep the game as a single self-contained HTML file unless the user asks for a restructure. Do not add a bundler, framework, or npm dependency without being asked.
- Keep the existing style: `const` and `let`, arrow functions, two-space indentation, and tunable values grouped in the top-level config objects (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `THEMES`).
- Put new tunable numbers in those config objects instead of hard-coding them in update or draw logic.
- Do not add external network requests, CDN scripts, or analytics. The game is fully offline.
- Preserve the `localStorage` key names above, and wrap any new storage access in a try/catch, since storage can be blocked in private or sandboxed contexts.
- Keep both input paths working (keyboard, mouse, and touch) when you change controls.
- Respect `reducedMotion` for any new screen shake, flash, or particle-heavy effect.

## Verifying changes

There is no automated test suite. To verify a change:

1. Open `index.html` in a browser (or serve it) and check the browser console for errors.
2. Start a game, then check the affected path: movement, firing, bombs, pause, stage transitions, boss fights, and game over.
3. Switch difficulty and mute, reload, and confirm the saved values persist.
4. For layout changes, check both a desktop-sized window and a phone-sized viewport.

If a headless browser is available, a Playwright smoke test that loads the page, clicks `#startBtn`, and asserts there are no console errors is a reasonable check.

## Git

- Development branch for this work: `claude/gifted-tesla-frefsh`. Do not push to other branches without explicit permission.
- Do not create pull requests unless the user asks.
