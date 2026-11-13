# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Google_4 is a single-page HTML5 canvas game, **Sky Fighter**: a Raiden-style vertical shoot-'em-up. The whole game lives in one file, `index.html`, with inline CSS and one inline JavaScript IIFE. There is no build step, no bundler, no package manager and no test suite.

Files:

- `index.html` (~1,670 lines): the entire game (markup, styles, and script).
- `README.md`: a one-line title only.
- `CLAUDE.md`: this file.

## Running and testing

- Open `index.html` directly in a browser. No server or install is needed.
- There are no automated tests or linters configured. Verify changes by playing the game in a browser, at both desktop and mobile viewport sizes.
- For headless checks, Playwright is available. Load `file:///.../index.html` and drive it with keyboard events. Use the Chromium at `/opt/pw-browsers/chromium` if a pinned version is needed.

## Architecture (inside `index.html`)

The script is wrapped in `(function () { ... })()`, so nothing leaks to `window`. Rough order of sections:

1. **DOM references**: `const x = document.getElementById(...)` for every HUD, overlay, and touch element. Edit the markup and these IDs together.
2. **Storage keys**: `STORAGE_BEST`, `STORAGE_MUTE`, `STORAGE_DIFF`, and `STORAGE_REDUCED`. All use the `skyfighter.*` prefix.
3. **Tuning tables**:
   - `DIFFICULTIES` (`easy` / `normal` / `hard`): spawn rates, enemy speed, shoot chance, starting lives and bombs, boss HP multiplier.
   - `KILLS_PER_STAGE` (18): kills needed before a stage boss spawns.
   - `THEMES`: six sky palettes, one per stage, cycled by stage number.
   - `ENEMY_TYPES`: `grunt`, `interceptor` (weaves), `tank` (4 HP). Each has a `minStage` gate.
   - `BOSSES`: six bosses (VANGUARD, BEHEMOTH, SPECTER, DREADNOUGHT, WRAITH, TITAN), each with colors and an `hpMul`.
4. **Audio**: a lazily created Web Audio context (`ensureAudio`). It uses `beep` and `noiseBurst` primitives, with named sound effects in `sfx`. Audio is created only after a user gesture.
5. **HUD and effects**: `renderHearts`, `renderBombs`, `flashHit`, `showComboPop`, `showStageBanner`, `screenShake`.
6. **Layout**: `BASE_W = 480`, `BASE_H = 720`. `resize()` scales the canvas to fit the window, capped at 1.6x.
7. **Input**: keyboard (`keys[e.code]`), mouse steering and click-to-fire, and a touch joystick plus FIRE and BOMB buttons.
8. **Game loop**: `update(dt)` runs each frame, followed by draw code. Key functions: `spawnBoss`, `useBomb`, and the stage-advance logic around `stage++`.

## Controls (keep the in-game hint in sync)

The hint is in `#controlsHint` in the markup.

- Mouse: move to steer, click to fire.
- Arrows / WASD: move. Space: fire.
- X or Shift: bomb. P or Esc: pause.
- Mobile: drag left to fly, tap FIRE, tap BOMB.

## Persistence

Only `localStorage` is used. Keys:

- `skyfighter.bestScore`: the high score.
- `skyfighter.muted`: `'1'` when muted.
- `skyfighter.difficulty`: `easy`, `normal`, or `hard`. Defaults to `normal`.
- `skyfighter.reducedMotion`: explicit user preference. If absent, the `prefers-reduced-motion` media query decides.

Any read or write must tolerate storage being unavailable (for example in private windows). Wrap new storage access in try/catch.

## Conventions

- Keep the single-file structure unless the user asks for a split. Do not add a build system, framework, or external dependencies without asking.
- Match the existing style: 2-space indentation, `const` and `let`, plain modern JavaScript with semicolons.
- Tuning changes should go in the constant tables (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `THEMES`), not be hard-coded into logic.
- The layout must work on phones: keep the `viewport-fit=cover` meta, the safe-area `env()` insets, and `touch-action: none` on the canvas and controls.
- Respect reduced motion: effects like screen shake and flashes should check the `reducedMotion` flag.
- Accessibility: announcements go through `announce()` and the `srLive` live region.

## Git workflow

- Work on the branch assigned for the session, and push there. Use clear, descriptive commit messages. Do not open pull requests unless asked.
- Do not commit generated or scratch files. The repo currently tracks only the files listed above.
