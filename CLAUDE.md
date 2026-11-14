# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a small static web project. Its only real content is **Sky Fighter**, a Raiden-style vertical shoot-'em-up built as a single self-contained HTML5 canvas page.

- `index.html` (~1,670 lines): the entire game. CSS is in a `<style>` block (starts ~line 10), markup is in the body (canvas `#gameCanvas` at ~line 363), and all game logic is in one IIFE inside a `<script>` block (starts ~line 422).
- `README.md`: a stub. It has no setup or usage instructions.

There is no build step, no package manager, no bundler, and no test suite. Nothing needs to be installed.

## Running the game

Open `index.html` directly in a browser, or serve the directory with any static server, for example:

```bash
python3 -m http.server 8000   # then visit http://localhost:8000
```

## Code layout inside `index.html`

The script is a single `(function () { 'use strict'; ... })();` closure. Nothing is exported or global. Sections, roughly in file order:

1. **DOM references**: `const x = document.getElementById(...)` for every HUD, overlay, and touch-control element.
2. **Storage keys**: `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion`.
3. **Tuning tables**: `DIFFICULTIES` (spawn rates, lives, bombs, boss HP multiplier), `KILLS_PER_STAGE`, `THEMES` (per-stage sky palettes), `ENEMY_TYPES`, `BOSSES`, `BOSS_PATTERNS`.
4. **Audio**: `ensureAudio()`, `beep()`, `noiseBurst()`, and a `sfx` object. Audio is synthesized with WebAudio, so there are no audio asset files.
5. **HUD and feedback helpers**: hearts/bombs rendering, combo pops, stage banners, screen shake, hit flash, and an `announce()` helper for screen readers.
6. **Input**: keyboard (`keys[e.code]`), a virtual joystick and fire/bomb buttons for touch, and mouse control on the canvas.
7. **Game state and flow**: `resetGame()`, `update(dt)`, `loseLife()`, `endGame()`, `togglePause()`, `useBomb()`, `fireWeapon()`, `applyPowerUp()`.
8. **Enemies and bosses**: spawn functions, `updateBoss()`, `killBoss()`, and the boss attack pattern functions (`fireRadial`, `fireAimedSpread`, `fireSpiralBurst`, and others).
9. **Rendering**: `draw()`, `drawPlane()`, `drawBossShip()`, `drawClouds()`. All of it is canvas 2D drawing.
10. **Main loop**: `loop(now)` calls `trackPerf`, then `update(dt)` and `draw()`. Frame time is clamped to 50 ms.

Key design points to keep in mind:

- **Fixed logical resolution**: `BASE_W = 480`, `BASE_H = 720`. `resize()` scales the canvas to fit the viewport. Game coordinates should always use the base size.
- **Adaptive quality**: `trackPerf()` steps a `quality` level (0 full, 1 medium, 2 low) based on average frame time over 45 frames. Effects should respect this level.
- **Reduced motion**: `reducedMotion` comes from the user's stored preference, or from `prefers-reduced-motion` if none is stored. It toggles the `reduced-motion` class on `<body>` and should gate screen shake and other motion effects.
- **Persistence**: best score, mute, difficulty, and motion preference are stored in `localStorage` under the `skyfighter.*` keys.

## Conventions

- Keep the game self-contained in `index.html`. Do not add a build system, frameworks, or external CDN dependencies unless the user asks for them.
- Follow the existing style: `const` and `let`, arrow functions for callbacks, 2-space indentation, and single quotes in JS.
- Put tuning numbers in the existing tables (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, and so on) rather than hard-coding them in logic.
- Keep the `'use strict'` IIFE wrapper. The code relies on closure-scoped state.
- Preserve accessibility hooks when changing UI: screen-reader updates go through `announce()`, which writes to the `#srLive` live region (`role="status"`, `aria-live="polite"`).

## Known caveats

- The `localStorage` reads at the top of the script (around lines 517 to 521) and the best-score write (around line 1477) are not wrapped in `try/catch`. Storage can throw in private browsing or in sandboxed iframes. Wrap these if the code is changed.
- The game has no automated tests. Verify changes by playing them in a browser, including keyboard, touch, and pause/resume.

## Git workflow

- Default development branch for this session: `claude/gifted-tesla-3h2ds3`. Commit and push there. Do not push to other branches without explicit permission.
- Do not create pull requests unless the user explicitly asks for one.
