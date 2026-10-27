# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo contains **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs in the browser.
The whole game is one self-contained file: `index.html` (about 1,670 lines) with inline `<style>` and an inline `<script>`. There is no framework, no bundler, and no external dependencies.

- `index.html`: the complete game (markup, CSS, and JavaScript).
- `README.md`: a placeholder. Update it if the project description changes.

## Running the game

No build step is needed. Open `index.html` directly in a browser, or serve the directory with any static server, for example:

```bash
python3 -m http.server 8000   # then visit http://localhost:8000/
```

There is no test suite, linter, formatter, or package manager configured. Verify changes by playing the game in a browser and checking the affected behavior. Check the browser console for errors.

## Code structure of `index.html`

The script is one IIFE (`'use strict'`) that runs on load. Sections are marked with `// ---------- Name ----------` comments. Follow the same style when adding code.

| Section | What it covers |
| --- | --- |
| DOM references | `const xEl = document.getElementById(...)` lookups at the top of the IIFE |
| Storage keys | `STORAGE_*` constants for `localStorage` |
| Adaptive performance | `trackPerf` lowers the `quality` level (0 full, 1 medium, 2 low) when frame time is high |
| Tuning data | `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES` (per-stage sky palettes), `ENEMY_TYPES`, `BOSSES` |
| Audio | `ensureAudio`, `beep`, `noiseBurst`, and the `sfx` table built from them (Web Audio, synthesized, no asset files) |
| HUD | `renderHearts`, `renderBombs`, `showComboPop`, `showStageBanner`, `screenShake` |
| Input | `keys` map, keyboard listeners, joystick and touch handling, mouse steering on the canvas |
| Game state | `resetGame`, `update(dt)`, `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`, `applyPowerUp`, `useBomb`, `fireWeapon` |
| Boss attacks | `BOSS_PATTERNS` (one array of attack functions per boss), `updateBoss`, `killBoss` |
| Rendering | `drawPlane`, `drawBossShip`, `drawClouds`, `draw()` |
| Main loop | `loop(now)` uses `requestAnimationFrame`, clamps `dt` to 0.05 s, and calls `update` then `draw` when running and not paused |

### Key conventions

- **Game state is module-level `let` variables** inside the IIFE. Keep new state there, and reset it in `resetGame()`.
- **Tuning lives in data tables** (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `THEMES`). Add or adjust content there rather than hard-coding numbers in logic.
- **Time-based movement.** Multiply per-frame changes by `dt` so the game runs at the same speed at any frame rate.
- **Controls.** Keyboard: arrows or WASD to move, Space to fire, X or Shift to bomb, P or Esc to pause. Mouse: move to steer, click to fire. Touch: a joystick on the left and FIRE and BOMB buttons on the right. Keep the `controlsHint` text in the overlay in sync with any control change.
- **Accessibility.** Announcements go through `announce()`, which writes to the `#srLive` live region. Honor reduced motion via `reducedMotion` and the `reduced-motion` body class. Preserve `aria-` attributes and `sr-only` text when editing markup.
- **Persistence.** Only `localStorage` is used, with keys prefixed `skyfighter.`: `bestScore`, `muted`, `difficulty`, and `reducedMotion`. Read and write them through the existing helpers, and keep the defaults when a key is missing.
- **Audio** starts only after a user gesture. Call `ensureAudio()` from input handlers, not at load time.

## Editing guidelines

- Keep the single-file structure unless the user asks for a split. Do not add a build system or dependencies without a clear reason.
- Match the surrounding code's naming, comment density, and indentation (two spaces, single quotes in JavaScript).
- Make small, focused changes. Don't refactor unrelated sections in the same change.
- Don't reference external asset files. Graphics are drawn with the canvas API and sound is synthesized.
- When you change a behavior that players can see (controls, difficulty, scoring, or bosses), mention it in the commit message and in the final summary.

## Git workflow

- Develop on the branch the task specifies. The default working branch in recent sessions is `claude/gifted-tesla-e53pu8`.
- Commit with clear, descriptive messages. Push with `git push -u origin <branch>`.
- Don't open a pull request unless the user asks for one.
