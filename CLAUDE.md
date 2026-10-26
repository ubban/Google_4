# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo is **Sky Fighter**, a single-file, browser-based vertical shoot-'em-up in the style of Raiden. The whole game lives in `index.html`: inline CSS in `<style>`, markup in `<body>`, and one inline `<script>` IIFE (`'use strict'`) that holds all game logic and rendering. `README.md` is a placeholder and has no setup instructions.

There is no build step, no package manager, no bundler, and no test suite. There is no `package.json`, no linter config, and no CI config.

## Running and verifying

- Open `index.html` directly in a browser, or serve the directory with any static server (for example `python3 -m http.server`).
- There are no automated checks. Verify changes by playing the game in a browser. Cover at least:
  - Keyboard and touch controls
  - Pause (P / Esc) and bomb (X / Shift)
  - Difficulty switching
  - Stage transitions and boss fights
  - Game over and best-score persistence
- For changes to layout or the HUD, also check a narrow (phone-width) viewport. The page uses touch controls when `ontouchstart` or `maxTouchPoints` is present.

## Code layout inside `index.html`

The script is organized by comment-headed sections. Keep new code in the matching section:

| Section | What it holds |
| --- | --- |
| DOM refs and `STORAGE_*` keys | `document.getElementById` lookups and `localStorage` key names (`skyfighter.*`) |
| Adaptive performance | `trackPerf` lowers `quality` when frame time is high |
| Constants | `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES`, `ENEMY_TYPES`, `BOSSES`, `BOSS_PATTERNS` |
| Audio | `ensureAudio`, `beep`, `noiseBurst`, and the `EXPLODE_SFX` mapping. Audio is synthesized with the Web Audio API, so there are no sound files. |
| HUD and feedback | `renderHearts`, `renderBombs`, `flashHit`, `showComboPop`, `showStageBanner`, `screenShake` |
| Difficulty and layout | `setDifficulty`, `resize` (fixed `BASE_W = 480`, `BASE_H = 720` canvas, scaled to fit) |
| Input | `keys` map, keyboard and pointer handlers, joystick, fire and bomb buttons |
| Game state | `resetGame`, `update(dt)`, `spawnEnemy`, `spawnBoss`, `applyPowerUp`, `useBomb`, `fireWeapon` |
| Boss patterns | `fire*` helpers and `BOSS_PATTERNS`, driven by `updateBoss` and `killBoss` |
| Rendering | `draw`, `drawPlane`, `drawBossShip`, `drawClouds` |
| Loop and wiring | `loop(now)` (dt clamped to 50 ms), start button handler, `requestAnimationFrame(loop)` |

The game is a single `requestAnimationFrame` loop. `update(dt)` runs only when the game is running and not paused, and `draw()` follows it.

## Conventions

- **Single file.** Do not split the game into modules or add a build step unless the user asks for it. Keep CSS in `<style>` and JS in the existing `<script>` block.
- **Style.** Match the surrounding code: 2-space indentation, single quotes in JS, arrow functions for callbacks, `const` by default, and `let` only when reassigned. Keep the short section-divider comments (`// ---------- Name ----------`).
- **Game data is data.** Add enemies, bosses, themes, or difficulty tiers by extending the constant tables rather than adding special-case branches.
- **Persistence.** Store user settings with the `skyfighter.*` keys in `localStorage`. Wrap new reads and writes in a way that still works when storage is unavailable, as the existing code does for best score.
- **Accessibility.** Respect `prefers-reduced-motion` (the `reducedMotion` flag and `reduced-motion` body class). Announce important state changes through `announce()` to the `#srLive` live region.
- **Performance.** The game targets 60 fps on modest devices. Avoid per-frame allocations in `update` and `draw` where a reused value works, and respect the `quality` level when adding expensive effects.
- **Comments.** Keep comments short and explain why something is non-obvious. Do not narrate the code.

## Git

- Develop on the branch named in the session instructions. Do not push to other branches without explicit permission.
- Commit messages are short, imperative, and describe the change. The existing history is `Initial commit` and `Add Sky Fighter: ...`.
- Do not open a pull request unless the user asks for one.
