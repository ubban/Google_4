# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Google_4 is a single-page browser game, **Sky Fighter**: a Raiden-style vertical shoot-'em-up rendered on an HTML `<canvas>`. The whole game lives in one file, `index.html` (about 1,670 lines: inline CSS, markup, and one inline `<script>`). `README.md` is only a placeholder.

There is no build step, package manager, bundler, test suite, or linter. To run the game, open `index.html` in a browser (or serve the directory with any static server).

## Repository layout

```
index.html   # Entire game: <style> (lines ~10-360), markup (~361-421), <script> (~422-1668)
README.md    # Placeholder
```

## Structure of `index.html`

- **Markup**: `#container` holds `#gameCanvas`, the HUD (`#hud`, `#score`, `#best`, `#mult`, `#weaponName`, `#weaponLvl`, `#hearts`, `#bombs`), the boss bar (`#bossBarWrap`), pop-up text (`#comboPop`, `#stageBanner`), touch controls (`#touchControls`), and the overlay menu (`#overlay`, with `#startBtn` and difficulty buttons `.diffBtn`).
- **Script**: wrapped in an IIFE with `'use strict'`. Nothing is exported to `window`. Sections are marked with `// ---------- Name ----------` comments, for example Adaptive performance, Input, and the game loop.
- **Config constants** (top of the script):
  - `DIFFICULTIES`: `easy` / `normal` / `hard`, with spawn rates, enemy speed, shoot chance, lives, bombs, and boss HP multiplier.
  - `KILLS_PER_STAGE` (18), `THEMES` (six sky palettes, one per stage), `ENEMY_TYPES` (`grunt`, `interceptor`, `tank`), `BOSSES` (six bosses), `BOSS_PATTERNS` (bullet-pattern functions).
- **Logical resolution**: the canvas is always `BASE_W x BASE_H` = 480 x 720. `resize()` only scales the CSS size. All game coordinates use this logical space.
- **Persistence**: `localStorage` keys are prefixed `skyfighter.` (`bestScore`, `muted`, `difficulty`, `reducedMotion`). Reads and writes are not wrapped in try/catch, so keep that in mind if you touch them.
- **Audio**: Web Audio API, created lazily by `ensureAudio()`. Sound effects are built from `beep()` and `noiseBurst()`. Sound is off until the first user gesture.
- **Game loop**: `requestAnimationFrame(loop)` calls `update(dt)` then `draw()`. `dt` is clamped to 0.05 s. `trackPerf()` adjusts a `quality` level (0-2) from the average frame time.

## Controls

| Input | Action |
| --- | --- |
| Mouse move / drag | Steer (handled by `canvasPointFromEvent`) |
| Mouse click, or hold the fire button | Fire |
| Arrow keys / WASD | Move |
| Space | Fire (`keys['Space']`) |
| X or Shift | Bomb (`useBomb()`) |
| P or Escape | Pause (`togglePause()`) |
| Touch | Left-side joystick, right-side FIRE and BOMB buttons |

Keyboard state is kept in the `keys` object, indexed by `KeyboardEvent.code`.

## Working conventions

- **Keep it in one file.** Do not split the game into modules or add a build step unless the user asks for it. Match the existing style: `const`/`let`, arrow functions in listeners, `function` declarations for game logic, and short section banners.
- **Game-state variables** (`player`, `bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`, `boss`, `score`, `lives`, `stage`, and so on) are declared with `let` in the IIFE. `resetGame()` is the single place that resets them for a new run, so add any new state there.
- **Code comments**: keep the sparse, section-level style of the existing file.
- **Accessibility**: the file respects `prefers-reduced-motion` (via the `reducedMotion` flag and the `reduced-motion` body class), uses `aria-live` (`#srLive`) for announcements through `announce()`, and has `.sr-only` text. Keep these working when you add features.
- **Mobile**: layout and controls must work at phone width. Touch input uses `passive: false` with `preventDefault()` on the buttons.

## Verifying changes

There are no automated tests or linters. Verify by:

1. Opening `index.html` in a browser and playing through at least one stage, a boss fight, and a game over.
2. Checking the browser console for errors.
3. Checking the touch layout with device emulation, if the change affects the HUD or controls.
4. For syntax-only checks, extracting the `<script>` block and running `node --check` on it works, since the code uses no modules.

## Git

- Development branch for this task: `claude/gifted-tesla-uj46jj`.
- Commit messages are short, imperative, and describe the change (see `git log`, for example "Add Sky Fighter: Raiden-style vertical shoot-em-up game").
- Do not open a pull request unless the user asks for one.
