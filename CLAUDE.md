# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a small, dependency-free front-end project. Its only real content is **Sky Fighter**, a Raiden-style vertical shoot-em-up built as a single self-contained HTML5 canvas page.

| File | Purpose |
| --- | --- |
| `index.html` | The whole game: markup, CSS (`<style>`), and JavaScript (`<script>`) in one file (~1,670 lines). |
| `README.md` | Minimal project readme. |

There is no build system, package manager, test suite, or linter configured yet.

## Running the project

No build step is needed. Open `index.html` directly in a browser, or serve the directory with any static server, for example:

```bash
python3 -m http.server 8000   # then visit http://localhost:8000/
```

## Code layout inside `index.html`

- **Markup** (`<body>`): a `<canvas id="gameCanvas">`, the HUD (score, best, multiplier, weapon, hearts, bombs), the start/pause/game-over overlay, the boss health bar, and touch controls (`#touchControls`, shown only on touch devices).
- **Styles** (`<style>`): all CSS is inline. Keep new styles here rather than adding stylesheet files.
- **Script** (`<script>`): a single IIFE with `'use strict'`. Nothing is exposed on `window`; keep new code inside the IIFE.
  - Constants and tuning tables near the top: `DIFFICULTIES`, `THEMES`, `ENEMY_TYPES`, `BOSSES`, `BOSS_PATTERNS`, `KILLS_PER_STAGE`, `BASE_W`/`BASE_H`.
  - Game state and loop: `resetGame()`, `update(dt)`, `draw()`, `loop(now)` (driven by `requestAnimationFrame`, with `dt` clamped to 50 ms).
  - Rendering helpers: `drawPlane`, `drawBossShip`, `drawClouds`, plus HUD helpers such as `renderHearts`, `renderBombs`, `showComboPop`, `showStageBanner`.
  - Audio: Web Audio API helpers `ensureAudio`, `beep`, `noiseBurst`. Audio starts only after a user gesture.
  - Input: keyboard (`keys` map), mouse on the canvas, and touch joystick plus fire/bomb buttons.
  - Adaptive quality: `trackPerf` lowers the `quality` level when frame times are slow.

## Gameplay controls

- Move: arrow keys or `WASD`, or mouse / touch drag.
- Fire: `Space` (held), or the fire button on touch devices.
- Bomb: `X` or `Shift`, or the bomb button.
- Pause: `P` or `Escape`.

## Persistence

Settings and scores are stored in `localStorage` under these keys. Keep the names stable, since changing them wipes players' saved data.

- `skyfighter.bestScore`
- `skyfighter.muted`
- `skyfighter.difficulty`
- `skyfighter.reducedMotion`

Reads and writes are not wrapped in `try/catch` today. If you add more storage access, guard it so the game still runs when storage is blocked (for example, in private browsing).

## Conventions

- Keep the game as a single self-contained file unless the user asks for a split. Do not add npm dependencies or external CDN scripts without a clear reason.
- Match the existing style: `const` and `let`, arrow functions for callbacks, two-space indentation, single quotes in JavaScript, and short section comments such as `// ---------- Input ----------`.
- Tuning values (speeds, spawn rates, HP, scores) live in the constant tables. Prefer changing those over hard-coding numbers inside update logic.
- Respect accessibility hooks already present: `reducedMotion` should suppress or soften screen shake and flashes, and state changes should be announced through `announce()` (the `#srLive` region).
- Keep the layout usable on phones. The canvas scales through `resize()` using `BASE_W` × `BASE_H` (480 × 720); avoid fixed pixel sizes that break on small screens.
- Do not commit generated or local files (editor settings, OS junk, screenshots) to the repository.

## Testing and verification

There are no automated tests. For changes to gameplay or rendering:

1. Load `index.html` in a browser and start a run on each difficulty.
2. Check the affected path directly: for example, fire and bombs for weapon changes, the boss fight for `BOSS_PATTERNS` changes, and the pause and game-over overlays for UI changes.
3. Check the layout at a narrow (phone) width and confirm touch controls appear on touch devices.
4. Confirm the browser console shows no errors.

If you add a test setup later, document the command here.

## Git workflow

- The default branch is `main`. Do feature work on a separate branch.
- Write clear, descriptive commit messages that explain why a change was made, not only what changed.
- Push with `git push -u origin <branch-name>`. Do not open a pull request unless the user asks for one.
