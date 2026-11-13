# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a small static web project. Its only real content is **Sky Fighter**, a Raiden-style vertical shoot-'em-up built as a single self-contained HTML5 canvas page.

| File | Purpose |
| --- | --- |
| `index.html` (~1,670 lines) | The whole game: inline `<style>`, markup for the HUD/overlays/touch controls, and one inline `<script>` containing the game engine. |
| `README.md` | One-line project title. |

There is no build system, package manager, test suite, or linter. Nothing needs to be installed.

## Running and testing

- Open `index.html` directly in a browser, or serve the directory with any static server (e.g. `python3 -m http.server`).
- There are no automated tests. Verify changes by playing the game in a browser, including the affected difficulty, weapon, boss, and touch paths when relevant.
- Keep the page working on phone-sized viewports (the viewport meta disables zoom and the layout uses `touch-action: none`).

## Architecture of `index.html`

Everything runs in one inline `<script>` (starting around line 422), wrapped in a single `(function () { ... })();` IIFE.

- **Rendering:** a single `<canvas id="gameCanvas">` driven by `requestAnimationFrame`. Game state is updated in `update(dt)` and drawn in `draw()`, with the main loop in `loop()`.
- **HUD and overlays:** DOM elements (score, best, multiplier, weapon, hearts, bombs, boss bar, stage banner, combo pop, pause/start overlay, vignette/flash) are updated from JS. They are not drawn on the canvas.
- **Persistence:** `localStorage` only, under the keys `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, and `skyfighter.reducedMotion`. Changing these key names orphans existing player data, so migrate deliberately if you ever rename them.
- **Difficulty:** the `DIFFICULTIES` table controls spawn and damage tuning. Selected via `.diffBtn` buttons and `setDifficulty()`.
- **Weapons:** `fireWeapon()` dispatches to pattern helpers (`fireAimedSpread`, `fireColumns`, `fireRadial`, `fireCross`, `fireDoubleRing`, `fireSpiralBurst`, `fireHomingSeeded`). Vulcan/Laser have three power levels.
- **Bombs:** limited smart bombs (`useBomb()`) clear the screen and damage bosses.
- **Stages and bosses:** stage-based waves (`spawnEnemy`, `showStageBanner`) end in a multi-pattern boss (`spawnBoss`, `updateBoss`, `killBoss`, `drawBossShip`) with a health bar.
- **Scoring:** `addScore()` with a kill-chain multiplier that `breakChain()` resets.
- **Power-ups:** `maybeSpawnPowerUp()` / `applyPowerUp()` (weapon level, bomb, extra life).
- **Audio:** Web Audio via `ensureAudio()`, `beep()`, and `noiseBurst()`, with stereo pan from `panFor()`. Audio is created lazily after a user gesture.
- **Input:** keyboard (arrows / WASD to move, Space to fire, X or Shift to bomb, P or Esc to pause), mouse/pointer, and touch (virtual joystick in `moveZone`, `fireBtn`, `bombBtn`).
- **Accessibility:** a reduced-motion toggle (`updateMotionBtn`) and a live region (`srLive`) for announcements via `announce()`.

## Conventions

- Keep the game as a **single self-contained file** unless the user asks to split it. Do not add a bundler, framework, or external dependencies without asking.
- Match the existing style: 2-space indentation, `const`/`let`, and plain functions.
- Keep game-balance numbers in the named constants or the `DIFFICULTIES` table rather than scattering magic numbers through the update loop.
- New `localStorage` keys should follow the `skyfighter.*` prefix.
- Preserve the touch and keyboard parity: any new action must be reachable on both input paths.
- Keep `update()` free of DOM writes where possible; DOM HUD updates belong in the existing render helpers (`renderHearts`, `renderBombs`, etc.).

## Git

- Development branch for automated work: `claude/gifted-tesla-w5lkwv`.
- Commit messages are imperative and descriptive, in the style of the existing history (e.g. "Add Sky Fighter: ...").
- Do not open pull requests unless explicitly asked.
