# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Sky Fighter is a Raiden-style vertical shoot-'em-up that runs in the browser. The whole game is one self-contained file, `index.html`: markup, CSS, and a single inline `<script>` that draws to a `<canvas>` with the 2D context and synthesizes all sound with the Web Audio API. There are no external dependencies, assets, or network calls.

Files:

- `index.html` – the entire game (about 1,670 lines).
- `README.md` – minimal project title.

## Running the game

There is no build step, package manager, or dev server. Open `index.html` directly in a browser, or serve the directory with any static file server. Verify changes by playing the game in a browser; there is no automated test suite.

## Code layout inside `index.html`

- **Markup and CSS** (roughly lines 1–420): HUD elements, overlay, touch controls, and styles. Elements are looked up by `id` in the script.
- **Script** (starts around line 422): one IIFE wrapped in `'use strict'`. Sections are marked with `// ---------- Name ----------` comments. Keep new code inside the IIFE; do not add globals.
- **Tuning data** (near the top of the script): `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES`, `ENEMY_TYPES`, and `BOSSES`. Change balance here rather than hard-coding numbers in logic.
- **Game logic**: `resetGame`, `spawnEnemy`, `spawnBoss`, `update(dt)`, `applyPowerUp`, `BOSS_PATTERNS` (bullet patterns), `updateBoss`, `killBoss`, `loseLife`, `endGame`, `togglePause`.
- **Rendering**: `draw()` and its helpers `drawPlane`, `drawBossShip`, `drawClouds`. `resize()` scales the fixed 480×720 canvas (`BASE_W`/`BASE_H`) to fit the window.
- **Audio**: `ensureAudio`, `beep`, `noiseBurst`, and the `sfx` table. Audio is created lazily on the first user gesture.
- **Main loop**: `loop(now)` calls `trackPerf`, then `update` and `draw`. `dt` is capped at 0.05 s.

## Controls

- Keyboard: arrows or WASD to move, Space to fire, X or Shift for a bomb, P or Esc to pause.
- Mouse: move to steer, click to fire.
- Touch: drag on the left to fly, tap FIRE, tap BOMB.

Key handling is in the keydown/keyup listeners near line 771. Keep the in-game hint text (`#controlsHint`) in sync when you change a binding.

## Persistence

The game stores four keys in `localStorage`. Preserve these names so existing players keep their data:

| Key | Meaning |
| --- | --- |
| `skyfighter.bestScore` | Highest score |
| `skyfighter.muted` | `'1'` when muted |
| `skyfighter.difficulty` | `easy`, `normal`, or `hard` |
| `skyfighter.reducedMotion` | `'1'` or `'0'`; falls back to the OS `prefers-reduced-motion` setting |

Wrap new storage access in care: `localStorage` can throw in private or restricted contexts, and the current code does not guard it.

## Performance and accessibility

- `trackPerf` adapts a `quality` level (0–2) from frame timing. New particle or draw effects should respect that level and the reduced-motion setting (`body.reduced-motion`).
- Screen-reader announcements go through `announce()`, which writes to the `#srLive` region. Use it for important state changes such as game over or a new stage.
- Keep the layout usable on phones: the canvas scales with the viewport, and touch controls appear for touch input.

## Conventions

- Vanilla JavaScript and CSS only. Do not add frameworks, bundlers, or CDN scripts unless the user asks.
- Match the existing style: 2-space indentation, single quotes, `const` by default, short section comments, and camelCase names. Constants that describe tuning are `UPPER_SNAKE_CASE`.
- Keep the game in one file unless the user asks for a split.
- Prefer small, focused edits. Do not reformat unrelated code.

## Verifying changes

Because there are no tests, check changes by hand in a browser:

1. Start a game, play through at least one stage boss, and confirm the stage banner and boss bar work.
2. Check all three difficulties and that the best score persists across a reload.
3. Test pause, mute, and the reduced-motion toggle.
4. Check touch controls at phone width (or in a device emulator) and make sure there is no horizontal scroll.
5. Open the browser console and confirm there are no errors.

## Git

- Develop on the branch the user specifies. Do not push to other branches without permission.
- Write clear commit messages that describe the change and its reason.
- Do not open a pull request unless the user asks for one.
