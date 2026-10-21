# CLAUDE.md

Guidance for Claude Code when working in this repository.

## What this repo is

`Google_4` is a small static web project. Its only real content is **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs in a browser canvas.

- `index.html` — the entire game: markup, CSS, and JavaScript in one file (~1,670 lines).
- `README.md` — a one-line title. Not user documentation.

There is no build step, package manager, bundler, test suite, linter, or CI configuration. Do not add tooling unless asked.

## Running it

Open `index.html` directly in a browser, or serve the directory with any static server (for example `python3 -m http.server`). Nothing needs to be installed.

## Code layout inside `index.html`

- **`<style>`** (lines 10–359): all styling, including the HUD, overlays, touch controls, and responsive layout.
- **`<body>`** (lines 361–421): HUD elements, the `<canvas id="gameCanvas">`, overlays, and the touch controls. Element IDs here are looked up by the script.
- **`<script>`** (starts around line 422): one IIFE with `'use strict'`. Everything is private to it; nothing is exported to `window`.

Within the script, the code is grouped by section comments:

| Area | What it holds |
| --- | --- |
| Config constants | `DIFFICULTIES`, `THEMES` (stage backgrounds), `ENEMY_TYPES`, `BOSSES`, `KILLS_PER_STAGE` |
| Persistence | `localStorage` keys prefixed `skyfighter.` (best score, mute, difficulty, reduced motion) |
| Audio | `ensureAudio`, `beep`, `noiseBurst`, and the `sfx` table. Web Audio is created lazily on the first user gesture. |
| Adaptive quality | `trackPerf` lowers or raises the `quality` level (0–2) from the average frame time |
| Input | keyboard, mouse, and touch joystick/buttons, all writing into shared state |
| Game state and loop | `resetGame`, `update(dt)`, `draw()`, and `loop(now)`, driven by `requestAnimationFrame` |
| Weapons, bombs, bosses | `fireWeapon`, `applyPowerUp`, the `fire*` pattern helpers, `BOSS_PATTERNS`, `updateBoss` |

The main loop caps `dt` at 0.05 s. Keep that cap when changing timing, so that a tab switch does not teleport objects.

## Conventions to follow

- **Match the existing style.** The file uses 2-space indentation, single quotes, `const`/`let`, arrow functions for callbacks, and short section banners like `// ---------- Name ----------`. Follow that when adding code.
- **Keep it in one file** unless the user asks for a split. The game is intentionally self-contained.
- **Tune gameplay through the constant tables** (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `THEMES`) rather than hard-coding numbers in logic.
- **Storage keys** must keep the `skyfighter.` prefix. Wrap new `localStorage` access so that a blocked or unavailable store does not break the game.
- **Accessibility:** respect `prefers-reduced-motion` (see `reducedMotion` and the screen-shake path) and keep the `srLive` announcements for important state changes.
- **Mobile:** the layout must work at phone width with touch controls. Check both keyboard/mouse and touch paths if you change input.

## Controls (for reference when testing)

- Mouse: move to steer, click or hold to fire.
- Arrows / WASD: move. Space: fire.
- X or Shift: bomb. P or Esc: pause.
- Touch: drag the left zone to fly, tap FIRE and BOMB.

## Working in this repo

- Work on the branch you were given. Do not push to other branches without permission.
- Verify changes by loading `index.html` in a browser and playing through the affected path. There are no automated tests to run.
- Keep commits focused. Describe the gameplay or UI effect in the message, not only the code change.
