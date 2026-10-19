# CLAUDE.md

Guidance for working in this repository.

## Project overview

Google_4 is a small static web project. Its only real content is **Sky Fighter**, a Raiden-style vertical shoot-'em-up rendered on an HTML5 `<canvas>`.

- `index.html`: the entire game. CSS is inline in `<style>`, markup is in `<body>`, and all game logic is in a single IIFE inside one `<script>` block.
- `README.md`: a stub with the project title only.

There is no build step, package manager, bundler, test suite, or linter. Nothing to install.

## Running it

Open `index.html` directly in a browser, or serve the directory with any static server, for example:

```sh
python3 -m http.server 8000
```

Then visit `http://localhost:8000/`.

## Architecture of `index.html`

The script is wrapped in `(function () { 'use strict'; ... })();`, so nothing leaks to `window`. Keep new code inside that wrapper.

Sections of the script, in source order:

1. **DOM references** (`const canvas = document.getElementById(...)` and similar). Every element the script uses is looked up once at the top. IDs must match the markup.
2. **Storage keys** (`STORAGE_*`), all prefixed `skyfighter.`.
3. **Adaptive performance** (`trackPerf`, `quality`). Frame times are averaged over 45 frames and `quality` steps between 0 (full), 1 (medium), and 2 (low) to keep frame rate up.
4. **Tuning tables**: `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES` (sky palettes per stage), `ENEMY_TYPES`, `BOSSES`. Most balance changes belong here rather than in logic code.
5. **Audio**: a Web Audio synth (`ensureAudio`, `beep`, `noiseBurst`, and an `sfx` object). Audio is created lazily on the first user gesture.
6. **HUD and effects**: `renderHearts`, `renderBombs`, `flashHit`, `showComboPop`, `showStageBanner`, `screenShake`.
7. **Input**: keyboard (`keys` map keyed by `KeyboardEvent.code`), touch joystick, touch and mouse fire/bomb buttons, and mouse steering on the canvas.
8. **Game state and loop logic**: `resetGame`, `update(dt)`, `spawnEnemy`, `spawnBoss`, `applyPowerUp`, `fireWeapon`, `useBomb`, `BOSS_PATTERNS`, `updateBoss`, `killBoss`, `loseLife`, `endGame`, `togglePause`.
9. **Rendering**: `draw()` and its helpers (`drawPlane`, `drawBossShip`, `drawClouds`).
10. **Main loop**: `loop(now)` clamps `dt` to 50 ms, calls `trackPerf`, then `update` and `draw` only when running and not paused. It is started with `requestAnimationFrame(loop)`.

### Conventions

- Time-based motion uses `dt` (seconds). Do not tie movement to frame count, except for the `frame` counter used for patterns.
- Settings and best score persist via `localStorage` under `skyfighter.*` keys. Every `localStorage` access is currently unguarded, so keep new storage reads in the same style and expect that private-mode or blocked-storage browsers may throw. If you touch persistence, consider wrapping it in `try/catch`.
- Respect the reduced-motion setting. It is applied in two places: JS checks the `reducedMotion` flag before screen shake and flashes, and CSS disables animations under `body.reduced-motion` (the `#vignette`, `#hearts`, `#comboPop`, `#stageBanner`, `#flash`, and overlay rules). New screen-wide effects should follow both patterns.
- Screen-reader announcements go through `announce(text)`, which writes to the `#srLive` live region.
- Keep the 480×720 logical play area (`BASE_W`, `BASE_H`). The `resize()` function scales the canvas to fit the viewport.
- Use the existing `DIFFICULTIES` and `THEMES` tables for new tuning values instead of hard-coding numbers in logic.
- Match the surrounding style: two-space indentation, single quotes in JS, `const` by default, and short comment headers like `// ---------- Name ----------` to separate sections.

## Testing and verification

There is no automated test setup. Verify changes by:

1. Loading `index.html` in a browser and playing through at least one stage.
2. Checking all three difficulties (Easy, Normal, Hard) if you touched tuning or spawn logic.
3. Testing on a narrow viewport or with touch emulation if you touched input, the HUD, or layout.
4. Checking the browser console for errors. The page should run with no uncaught exceptions.

If you add a test harness or a linter, record the commands here.

## Making changes

- Work on the designated feature branch. Do not push to `main` directly.
- Keep commits focused. A commit that changes game balance should not also restyle the HUD.
- Because the game is one file, diffs can be large. Prefer targeted edits over rewrites of whole sections.
- Update this file when you add a new section to the script, a new storage key, a new tuning table, or a new build or test step.
