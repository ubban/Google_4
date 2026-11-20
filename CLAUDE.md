# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo contains **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs in the browser. The entire game is one self-contained file: `index.html`, with inline `<style>` and an inline `<script>`.

- No build step, bundler, package manager, or external dependencies.
- No test suite, linter, or CI configuration exists yet.
- `README.md` is a placeholder and currently only contains the project name plus a stray timestamp line.

## Running the game

Open `index.html` directly in a browser, or serve the directory with any static file server, for example:

```bash
python3 -m http.server 8000   # then visit http://localhost:8000/
```

## Code layout (`index.html`)

- **Head/meta**: viewport is locked (`user-scalable=no`, `viewport-fit=cover`). The favicon is an inline SVG data URI.
- **`<style>`** (about lines 10–362): all styling, including the HUD, overlay, and touch controls.
- **Markup** (about lines 363–421): `<canvas id="gameCanvas">`, the overlay, HUD elements, the touch joystick, and the fire/bomb buttons.
- **`<script>`** (from about line 422): a single IIFE wrapping all game logic, in `'use strict'` mode. Sections, in order:
  - DOM references and constants: `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES`, `ENEMY_TYPES`, `BOSSES`
  - Persistence keys: `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion`
  - Audio: Web Audio helpers `beep`, `noiseBurst`, `ensureAudio`, `panFor`
  - HUD and effects: `renderHearts`, `renderBombs`, `flashHit`, `showComboPop`, `showStageBanner`, `screenShake`, `announce` (screen-reader live region)
  - Layout: `BASE_W = 480`, `BASE_H = 720`, `resize()` scales the fixed logical canvas to the viewport
  - Input: keyboard handlers, a touch joystick, and mouse/pointer helpers
  - Game state and loop: `resetGame`, `update(dt)`, `draw()`, `loop(now)`. `loop` clamps `dt` to 0.05 s and runs `requestAnimationFrame`.
  - Enemies, bosses, and weapons: `spawnEnemy`, `spawnBoss`, `BOSS_PATTERNS`, `updateBoss`, `fireWeapon`, `useBomb`
  - Lifecycle: `loseLife`, `killBoss`, `endGame`, `togglePause`
  - Adaptive quality: `trackPerf` lowers or raises a `quality` level based on average frame time over 45 frames.

## Controls

| Action        | Keyboard                              | Touch                    |
|---------------|---------------------------------------|--------------------------|
| Move          | Arrow keys or WASD                    | Left-side joystick       |
| Fire          | Space (hold)                          | Fire button (hold)       |
| Smart bomb    | X, or Shift                           | Bomb button              |
| Pause         | P or Escape                           | Pause button             |

## Game rules and tuning

- **Difficulty** (`DIFFICULTIES`): easy, normal, and hard. Each one sets spawn rate, enemy speed, enemy shoot chance, starting lives, starting bombs, and boss HP multiplier. Changing a difficulty row changes gameplay for everyone.
- **Stages**: each stage ends after `KILLS_PER_STAGE` (18) kills and brings a boss. `BOSSES` and `THEMES` cycle by stage using modulo (`(stage - 1) % length`), and `BOSS_PATTERNS` defines bullet patterns.
- **Power-ups** are dropped by `maybeSpawnPowerUp` and applied by `applyPowerUp`. Weapon level is shown in the HUD.
- **Combos** are tracked by `addScore`, `breakChain`, and the multiplier shown in `#mult`.

## Persistence

State is stored in `localStorage` under the `skyfighter.*` keys listed above. Reads and writes are not wrapped in `try/catch`, so a browser with storage disabled can throw. If you touch this code, consider adding guards.

Reduced motion honors the `prefers-reduced-motion` media query by default. Users can override it with the motion toggle, which is saved to `skyfighter.reducedMotion`.

## Conventions when editing

- Keep the game in a single file unless the user asks for a split. Do not add a build step or dependencies without a reason.
- Preserve the single IIFE and `'use strict'` wrapper. Game code should remain private to the IIFE and not leak onto `window`.
- Game logic uses delta-time (`dt`, in seconds). Keep new movement and timers frame-rate independent.
- Accessibility features (`announce`/`srLive`, reduced motion, `aria-*` attributes) should be kept working when you change related UI.
- Keep the mobile layout working. The canvas scales via `resize()`, and touch controls must stay usable at phone widths.

## Verifying changes

There is no automated test suite. For a change to gameplay or UI:

1. Load `index.html` in a browser and check the browser console for errors.
2. Play through start, pause, game over, and restart. Confirm the best score persists across reloads.
3. Check the touch controls at a phone-sized viewport if the change touches input or layout.

For visual or interactive checks, a headless Chromium with Playwright works well. Load the page with `file://` or the static server above.

## Git

- Development branch: `claude/gifted-tesla-m3yx4b`. Push there with `git push -u origin <branch>`.
- Do not open a pull request unless the user asks for one.
