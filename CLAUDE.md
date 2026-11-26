# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a small, dependency-free static web project. Its only real content is **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs in a single HTML page on a `<canvas>`.

## Repository layout

```
.
├── index.html   # The entire game: markup, CSS, and JavaScript in one file (~1,670 lines)
├── README.md    # Minimal placeholder (title only)
└── CLAUDE.md    # This file
```

There is no build step, package manager, test suite, linter, or CI configuration. Do not add one unless asked.

## Running the game

No build is needed. Open `index.html` directly in a browser, or serve the directory with any static server, for example:

```bash
python3 -m http.server 8000   # then visit http://localhost:8000/
```

## Architecture of `index.html`

Everything lives in one file, in this order:

1. **`<head>`**: meta tags (viewport locked for mobile, `theme-color`), an inline SVG favicon, and the page title.
2. **`<style>`**: CSS with `color-scheme: dark`. Colors come from a fixed palette (`#05122b` background, `#0b1d3a` theme). The HUD is absolutely positioned and respects `env(safe-area-inset-*)`. `body` uses `touch-action: none` and `user-select: none` so the game owns touch input.
3. **`<body>`**: a `#gameCanvas`, a HUD (score, best, multiplier, weapon, hearts, bombs, boss bar), an overlay for the title, pause, and game-over screens, touch controls (joystick zone, FIRE, BOMB), and an `aria-live` region (`#srLive`) for screen-reader announcements.
4. **`<script>`**: one IIFE (`'use strict'`) containing all game logic. Nothing is exported to `window`.

### Key pieces inside the script

| Area | Where to look | Notes |
|---|---|---|
| Persistence | `STORAGE_*` constants | `localStorage` keys prefixed `skyfighter.` (best score, mute, difficulty, reduced motion). Reads and writes are not wrapped in try/catch. |
| Difficulty | `DIFFICULTIES` | `easy` / `normal` / `hard`. Controls spawn rates, enemy speed and fire chance, starting lives and bombs, and boss HP multiplier. |
| Stages | `KILLS_PER_STAGE`, `THEMES`, `BOSSES` | A stage advances every 18 kills. Each stage has a sky theme and a boss. Boss patterns cycle through `BOSS_PATTERNS`. |
| Enemies | `ENEMY_TYPES`, `spawnEnemy` | `grunt`, `interceptor` (weaving), and `tank` (4 HP, appears from stage 2). |
| Player weapons | `fireWeapon`, `applyPowerUp` | Power-ups change the weapon type and level. |
| Bombs | `useBomb` | Clears bullets and damages enemies. |
| Update / draw | `update(dt)`, `draw()`, `loop(now)` | Fixed-ish timestep driven by `requestAnimationFrame`. `dt` is capped at 50 ms. `update` runs only when `running && !paused`. |
| Adaptive quality | `trackPerf`, `quality` | Rolling 45-frame average lowers or raises a quality level (0 full, 1 medium, 2 low) based on frame time. |
| Audio | `ensureAudio`, `beep`, `noiseBurst` | Web Audio synthesis only, with no audio files. `ensureAudio()` must run after a user gesture. |
| Input | `keydown` / `keyup`, touch handlers, `canvasPointFromEvent` | Keyboard, mouse, and touch are all supported. |
| Accessibility | `announce`, `reducedMotion`, `prefers-reduced-motion` | Reduced motion is honored by default and can be toggled. Screen-reader updates go through `#srLive`. |

### Controls (as implemented)

- **Move:** Arrow keys or WASD, mouse (steer), or left-side drag/joystick on touch.
- **Fire:** Space (hold), mouse click, or the FIRE button.
- **Bomb:** `X`, either Shift, or the BOMB button.
- **Pause:** `P` or `Esc`.

## Conventions

- **Keep it a single self-contained file** unless the user asks to split it. Do not introduce a bundler, framework, or external CDN dependency.
- **Preserve the IIFE and `'use strict'`.** Do not leak globals.
- **Use the existing constants.** Balance changes belong in `DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `THEMES`, and `BOSS_PATTERNS`, not scattered as magic numbers.
- **Keep the storage key prefix** (`skyfighter.`) so existing player saves keep working.
- **Respect the accessibility hooks.** New on-screen feedback should consider `reducedMotion` and, where useful, call `announce()`.
- **Mobile first.** Keep `touch-action: none`, the safe-area insets, and the fixed-viewport meta tag intact. Test touch controls when changing input code.
- **Match the existing style:** two-space indentation, single quotes in JS, and short comment headers (`// ---------- Section ----------`) to separate sections.

## Making changes

- Read the relevant section of `index.html` before editing. Line numbers shift, so search by identifier.
- Because there are no tests, verify gameplay changes by loading the page in a browser and playing through the affected path (start, fire, bomb, pause, game over, and the stage/boss path if relevant).
- Keep commits focused. Use a clear imperative subject line that describes the gameplay or code change.

## Things to be aware of

- `README.md` is only a placeholder title with a stray timestamp. Update it only if the user asks for real project documentation.
- Game state is entirely client-side. Best score and settings are stored in the browser's `localStorage`, not on a server.
