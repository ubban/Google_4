# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a small static web project. Its only real content is **Sky Fighter**, a
single-page, Raiden-style vertical shoot-'em-up built on an HTML5 `<canvas>`.

| File         | Purpose                                                                 |
|--------------|-------------------------------------------------------------------------|
| `index.html` | The entire game: markup, CSS (`<style>`), and JavaScript (`<script>`).  |
| `README.md`  | Minimal project readme (currently just the title and a timestamp line). |

There is no build system, package manager, bundler, test suite, or server-side code.

## Running the game

Open `index.html` directly in a browser, or serve the directory with any static server, e.g.:

```bash
python3 -m http.server 8000   # then visit http://localhost:8000/
```

No install or compile step is required.

## Code layout inside `index.html`

- **`<head>`**: meta tags, theme color, and an inline SVG favicon (data URI).
- **`<style>`**: all CSS, using a dark color scheme. Includes the pill-style HUD, glassmorphic
  overlay card, combo/stage banners, low-health vignette, hit flash, and touch-control layout.
  Respects `body.reduced-motion`.
- **Body markup**: canvas (`#gameCanvas`), HUD elements, overlay/title card, touch controls
  (`#moveZone`, `#fireBtn`, `#bombBtn`), mute/motion/pause buttons.
- **`<script>`**: one IIFE (`'use strict'`) containing the whole game. Major sections, in order:
  1. DOM references and `STORAGE_*` keys.
  2. `DIFFICULTIES` table, `KILLS_PER_STAGE`, `ENEMY_TYPES`, `BOSSES`, weapon/power-up definitions.
  3. Persisted settings loaded from `localStorage` (best score, mute, difficulty, reduced motion).
  4. Audio: `ensureAudio`, `beep`, `noiseBurst`, and the `sfx` object (Web Audio API).
  5. HUD rendering and effects (`renderHearts`, `renderBombs`, `flashHit`, `showComboPop`,
     `showStageBanner`, `screenShake`).
  6. Input: keyboard handlers, virtual joystick and fire/bomb buttons for touch, mouse on canvas.
  7. Game state: `resetGame`, `update(dt)`, spawning (`spawnEnemy`, `spawnBoss`,
     `maybeSpawnPowerUp`), collisions (`rectsOverlap`), scoring/combo (`addScore`, `breakChain`).
  8. Weapons and bosses: `fireWeapon`, `applyPowerUp`, boss bullet patterns (`fireRadial`,
     `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`, `fireCross`, `fireHomingSeeded`,
     `fireDoubleRing`), `updateBoss`, `killBoss`.
  9. Lifecycle: `loseLife`, `endGame`, `togglePause`.
  10. Rendering: `drawPlane`, `drawBossShip`, `drawClouds`, `draw()`.
  11. Main loop: `loop(now)` driven by `requestAnimationFrame`; start button wiring at the end.

### Controls (for reference when testing)

- **Move**: arrow keys or WASD (handled in `update()`), mouse on canvas, or touch joystick.
- **Fire**: hold Space or the fire button (mouse or touch).
- **Bomb**: `X` or `Shift`, or the bomb button. Bombs clear the screen and damage bosses.
- **Pause**: `P` or `Escape`, or the pause button.

## Conventions

- **Keep the game self-contained.** Do not add external scripts, stylesheets, or fonts unless
  the user asks. The page is intentionally a single file that works offline.
- **Preserve the single-IIFE structure.** New game logic belongs inside the existing IIFE;
  don't leak globals onto `window`.
- **Use the existing constants and tables.** Tune difficulty via `DIFFICULTIES`, stage pacing
  via `KILLS_PER_STAGE`, and enemies/bosses via `ENEMY_TYPES` / `BOSSES` rather than hard-coding
  numbers in logic.
- **Persisted state uses the `skyfighter.*` prefix** in `localStorage`. Wrap any new storage
  access in a way that tolerates it being unavailable (private mode, blocked storage).
- **Accessibility and input parity.** Any new interaction should work with keyboard, mouse,
  and touch. Respect `reduced-motion` for new animations, and announce important state
  changes through the existing `srLive` live region where appropriate.
- **Mobile first.** Layout must work at phone widths; the canvas scales via `resize()`.
  Keep `touch-action: none` and the safe-area insets intact.
- **Frame-rate independence.** Game updates take a `dt`; scale movement and timers by it.
- **Match existing style.** Two-space indentation, single quotes in JS, `const`/`let`, arrow
  functions for callbacks, and concise comments that explain *why* rather than *what*.

## Working in this repo

- Default branch work happens on the branch named in the session instructions; do not push to
  other branches without explicit permission.
- Commit messages should be short imperative summaries, with a body for non-trivial changes
  (the existing history is descriptive, e.g. "Add Sky Fighter: ...").
- There are no automated tests or linters. Verify gameplay changes by loading `index.html` in a
  browser (desktop and a narrow/mobile viewport) and exercising the affected flow: start,
  fire, bomb, pause, stage transition, boss fight, and game over.
- Do not commit generated artifacts, screenshots, or scratch files to the repo. Use the
  session scratchpad for temporary output.

## Notes for future updates

Keep this file accurate when the structure changes. In particular, update the
"Code layout" section if the game is split into multiple files, if a build step is added, or if
a test or lint command is introduced (add the exact command under a new "Commands" heading).
