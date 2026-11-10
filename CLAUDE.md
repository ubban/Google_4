# CLAUDE.md

Guidance for Claude Code when working in this repository.

## What this project is

Google_4 is a small static web project. Its only real content is **Sky Fighter**, a
Raiden-style vertical shoot-em-up game, implemented as a single self-contained HTML
file (`index.html`, ~59 KB) using HTML5 Canvas 2D, inline CSS, and inline JavaScript.
`README.md` is a placeholder with no setup instructions.

## Repository layout

```
index.html   # The entire game: markup, CSS (<style>), and JS (<script>)
README.md    # Placeholder
CLAUDE.md    # This file
```

There is no build system, package manager, bundler, test suite, linter, or CI config.
Do not invent `npm`/build/test commands. To run the game, open `index.html` in a
browser (or serve the directory with any static server).

## Structure of index.html

- **`<style>` (roughly lines 10–360):** HUD pills, hearts/bombs, glassmorphic overlay
  card, combo/stage banners, vignette and hit flash, touch controls, boss health bar.
- **`<body>` (roughly lines 363–421):** `#gameCanvas`, overlay/title/stats elements,
  HUD elements, and `#touchControls` (joystick, fire, and bomb buttons).
- **`<script>` (from line 422):** one IIFE (`'use strict'`) containing all game logic.
  Sections, in order:
  - DOM references and `STORAGE_*` keys (`skyfighter.*` in `localStorage`)
  - Tuning tables: `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES`, `ENEMY_TYPES`, `BOSSES`,
    `BOSS_PATTERNS`
  - Audio: `ensureAudio`, `beep`, `noiseBurst` (Web Audio, created lazily on first gesture)
  - Rendering helpers: `drawPlane`, `drawBossShip`, `drawClouds`, `draw`
  - Input: keyboard (`keys[e.code]`), mouse/pointer, and touch joystick/buttons
  - Simulation: `update(dt)`, `spawnEnemy`, `spawnBoss`, `fireWeapon`, `useBomb`,
    `applyPowerUp`, `updateBoss`, `killBoss`, `loseLife`, `endGame`
  - Loop: `loop(now)` drives `update` then `draw` via `requestAnimationFrame`; `dt` is
    clamped to 50 ms.

## Game behavior to preserve

- **Controls:** arrows/WASD move; Space or held pointer fires; `X`/`Shift` drops a bomb;
  `P`/`Esc` pauses. Touch devices get on-screen joystick, fire, and bomb controls.
- **Persistence:** best score, mute, difficulty, and reduced-motion preference are saved
  to `localStorage` under `skyfighter.*`. Reads and writes are already tolerant of
  missing values (`|| default`); keep them that way.
- **Difficulty:** `easy` / `normal` / `hard` change spawn rate, enemy fire chance, lives,
  starting bombs, and boss HP multiplier.
- **Progression:** `KILLS_PER_STAGE` kills advance the stage; each stage has a theme and
  eventually a multi-pattern boss with a health bar.
- **Scoring:** kill-chain multiplier that resets on `breakChain()`.
- **Weapons:** Vulcan (3 spread levels) and Laser; power-ups raise weapon level, grant
  bombs, or extra lives.
- **Adaptive quality:** `trackPerf` lowers/raises `quality` based on a 45-frame rolling
  average of frame time.
- **Accessibility:** respects `prefers-reduced-motion` and has screen-reader announcements
  via `#srLive` and `announce()`. Don't remove these when editing UI.

## Conventions

- Keep the game as a **single file** unless the user asks for a split. Don't add a
  framework, bundler, or external dependency without an explicit request.
- Match the existing style: 2-space indentation, `const`/`let`, arrow functions for
  callbacks, `camelCase` for functions and variables, `UPPER_SNAKE_CASE` for constants
  (`BASE_W`, `KILLS_PER_STAGE`, `STORAGE_BEST`).
- Game-logic values belong in the tuning tables near the top of the script, not
  scattered as magic numbers, when they are meant to be tuned.
- Use `dt` (seconds) for all time-based motion so behavior is frame-rate independent.
- Keep the canvas at its logical size (`BASE_W` × `BASE_H` = 480 × 720) and let
  `resize()` scale it for the viewport.
- Don't add new global variables; keep state inside the IIFE.

## Verifying changes

Since there are no automated tests, verify by opening `index.html` in a browser and
checking the affected behavior by hand. For UI or gameplay changes, also check:

- Desktop keyboard controls and the touch layout (for example, with device emulation)
- Pause/resume, game over, and restart
- Best-score persistence after a reload
- Difficulty switching and mute toggling

If you can't run a browser in the current environment, say so explicitly in your summary
rather than claiming the behavior was tested.

## Git

- Development branch for this work: `claude/gifted-tesla-o4qlyj`.
- Commit messages: imperative subject line, followed by a short body that explains the
  "why" when it isn't obvious from the diff.
- Don't open a pull request unless the user asks for one.
