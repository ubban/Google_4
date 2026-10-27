# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Google_4 is a small static web project containing **Sky Fighter**, a Raiden-style
vertical shoot-'em-up rendered on an HTML5 `<canvas>`. The whole game lives in one
file, `index.html`: markup, CSS, and JavaScript are all inline. `README.md` is a
placeholder with only the project title.

There is no package manager, build step, bundler, test suite, linter, or CI config.
Open `index.html` directly in a browser (or serve the directory with any static file
server) to play.

## Repository layout

```
index.html   # The entire game: <style> (~lines 10-359), markup (~362-421), <script> (~422-1667)
README.md    # Project title only
CLAUDE.md    # This file
```

Inside `index.html`, the script is a single IIFE with `'use strict'`. Everything is
closed over; nothing is exported or attached to `window`. Keep it that way unless
there is a reason to expose something.

## Code structure (inside the script)

Sections appear roughly in this order, marked with `// ---------- Name ----------` comments:

- **DOM refs and constants**: `canvas`, `ctx`, element lookups, `STORAGE_*` keys.
- **Adaptive performance**: `trackPerf` drops the `quality` level (0 full, 1 medium,
  2 low) when frame time is poor.
- **Tuning tables**: `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES` (sky palettes per
  stage), `ENEMY_TYPES`, `BOSSES`. Balance changes usually belong here.
- **Audio**: `ensureAudio`, `beep`, `noiseBurst`, and `EXPLODE_SFX`. Audio uses the
  Web Audio API and is created lazily on the first user gesture.
- **HUD and feedback**: `renderHearts`, `renderBombs`, `flashHit`, `showComboPop`,
  `showStageBanner`, `screenShake`, `announce` (screen-reader live region).
- **Layout and input**: `resize` (scales a `BASE_W` × `BASE_H` = 480 × 720 logical
  canvas), joystick and touch handling, mouse/pointer handling, and keyboard handling.
- **Game state**: `resetGame`, `update(dt)`, `spawnEnemy`, `spawnBoss`,
  `maybeSpawnPowerUp`, `applyPowerUp`, `useBomb`, `fireWeapon`, `addScore` / `breakChain`
  (kill-chain multiplier), `loseLife`, `endGame`, `togglePause`.
- **Boss patterns**: `BOSS_PATTERNS` is an array of bullet-pattern functions
  (`fireRadial`, `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`, `fireCross`,
  `fireHomingSeeded`, `fireDoubleRing`) used by `updateBoss`.
- **Rendering**: `drawPlane`, `drawBossShip`, `drawClouds` (parallax), `draw()`.
- **Main loop**: `loop(now)` clamps `dt` to 0.05 s, runs `update` and `draw` only when
  running and not paused, then calls `requestAnimationFrame`.

Game objects are plain arrays and objects updated in place each frame. Collision uses
the `rectsOverlap` helper.

## Persistence

`localStorage` keys, all prefixed `skyfighter.`:

| Key | Holds |
| --- | --- |
| `skyfighter.bestScore` | Best score (number) |
| `skyfighter.muted` | `'1'` when muted |
| `skyfighter.difficulty` | `easy`, `normal`, or `hard` |
| `skyfighter.reducedMotion` | `'1'` or `'0'`; if unset, follows `prefers-reduced-motion` |

Reads and writes are not wrapped in try/catch. If you add new storage, guard it so the
game still works when storage is blocked (private windows, disabled site data).

## Conventions

- **Single file.** Add new code to `index.html` unless the user asks for a split. Do not
  introduce a build step or external dependencies without asking.
- **Match the surrounding style.** Two-space indentation, single quotes in JS, `const`
  by default, arrow functions for callbacks, and the existing section comment banners.
- **Tuning values are inline numbers.** Keep them in the existing tables and objects
  rather than scattering magic numbers.
- **Accessibility and input parity.** Any new control needs keyboard, mouse, and touch
  paths. Respect `reducedMotion` for new visual effects such as shake or flashes, and
  announce important state changes through `announce()`.
- **Responsive layout.** Mobile use is a priority. Check changes at a phone-sized
  viewport, and keep the `touchControls` overlay from covering gameplay.
- **Comments.** Explain why something is non-obvious. Do not narrate what the code does.

## Verifying changes

No automated checks exist, so verify by hand:

1. Open `index.html` in a browser, or serve the directory with `python3 -m http.server`.
2. Play through at least one stage, a boss fight, a power-up pickup, a bomb, and a
   game over. Check the score, best score, and difficulty persistence after reloading.
3. Toggle pause (P or Esc), mute, and reduced motion.
4. For layout or input changes, test at a narrow mobile width with touch emulation.
5. Check the browser console for errors.

If you can run a headless browser (Playwright/Chromium), a scripted smoke test is a
reasonable way to exercise the main loop, but do not add test tooling to the repo
without asking.

## Git

- Develop on the branch the session specifies. Do not push to `main` directly.
- Keep commits focused, and write messages that explain the change and its reason.
- Do not open a pull request unless the user asks for one.
