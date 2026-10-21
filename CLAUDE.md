# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo is a small, dependency-free browser game: **Sky Fighter**, a Raiden-style vertical shoot-em-up. The whole game lives in one file, `index.html` (~1,670 lines), containing inline CSS, markup, and a single IIFE-wrapped `<script>`. There is no build step, no package manager, no test suite, and no external assets. Audio is synthesized with the Web Audio API and all graphics are drawn on a 2D canvas.

Other files:

- `README.md` — a stub with only the title. Keep it short; project docs belong here only if the user asks.
- `.git/` — the repo history (`Initial commit`, then the Sky Fighter commit).

## Running the game

No install is needed. Open `index.html` directly in a browser, or serve the directory with any static server, for example:

```sh
python3 -m http.server 8000
```

Then visit `http://localhost:8000/index.html`.

## File layout of `index.html`

| Section | Approx. lines | Contents |
| --- | --- | --- |
| `<style>` | 10–360 | Layout, HUD pills, glass overlay card, touch controls, animations, reduced-motion rules |
| `<body>` | 361–421 | Canvas `#gameCanvas`, HUD elements, overlay, touch controls, difficulty buttons |
| `<script>` | 422–1667 | All game logic, wrapped in `(function () { 'use strict'; ... })()` |

Within the script, the code is organized roughly as:

1. **DOM references** — `const xxxEl = document.getElementById(...)` at the top. Add new elements the same way.
2. **Constants and config** — `STORAGE_*` keys, `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES` (per-stage sky palettes), `ENEMY_TYPES`, `BOSSES`.
3. **Persistence and settings** — best score, mute, difficulty, and reduced-motion flags, all in `localStorage`.
4. **Audio** — `ensureAudio()` lazily creates the `AudioContext` (browsers require a user gesture), plus `beep()`, `noiseBurst()`, and named SFX.
5. **HUD and feedback helpers** — hearts, bombs, combo pop, stage banner, screen shake, hit flash.
6. **Input** — keyboard (`keys[e.code]`), mouse/pointer position, and touch joystick/fire/bomb controls.
7. **Game state** — `resetGame()`, `update(dt)` (movement, firing, collisions, spawning, scoring), `applyPowerUp()`.
8. **Bosses** — `BOSS_PATTERNS` (radial, aimed spread, columns, spiral, cross, homing, double ring), `updateBoss()`, `killBoss()`.
9. **Rendering** — `drawPlane()`, `drawBossShip()`, `drawClouds()`, and `draw()`.
10. **Main loop** — `loop(now)` clamps `dt` to 0.05 s, calls `trackPerf`, then `update` and `draw` while running and unpaused.

## Game mechanics reference

- **Controls:** arrows or WASD to move; Space or hold FIRE to shoot; `X` or Shift (or the BOMB button) for a smart bomb (limited per difficulty); `P` or `Escape` to pause. On touch devices, a joystick on the left, plus FIRE and BOMB buttons.
- **Difficulty** (`DIFFICULTIES`): Easy, Normal (default), Hard. Each sets spawn rate, enemy speed, enemy fire chance, starting lives, starting bombs, and boss HP multiplier.
- **Stages:** every `KILLS_PER_STAGE` (18) kills advances the stage. Stages cycle through `THEMES`. Tanks appear from stage 2 (`minStage`).
- **Enemies:** `grunt`, `interceptor` (fast, weaving), `tank` (4 HP).
- **Bosses:** one per stage, cycling through `BOSSES`, with multi-pattern attacks and a health bar.
- **Power-ups:** `life`, `bomb`, `weapon` (Vulcan/Laser, 3 levels).
- **Scoring:** kill-chain combo multiplier (`addScore`, `breakChain`).
- **Persistence:** `localStorage` keys `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion`. Keep these keys stable, since changing them wipes players' saved state.

## Conventions

- **Single file.** Keep everything in `index.html` unless the user asks for a split. Do not add a bundler, framework, or npm dependencies.
- **Style:** the existing code uses 2-space indentation, single quotes in JS, `const` by default, arrow functions for callbacks, and `'use strict'` inside the IIFE. Match this style. Keep comments sparse, matching the surrounding code.
- **Naming:** `camelCase` for functions and variables, `UPPER_SNAKE_CASE` for constants and config tables, `STORAGE_*` for `localStorage` keys, and `*El` suffix for cached DOM elements.
- **Frame-rate independence:** game logic must scale by `dt` (seconds). Do not step per frame.
- **Accessibility:** respect the reduced-motion toggle (`reducedMotion`) for any new screen shake, flash, or pulsing effect. Announce important state changes through `announce()` (the `srLive` live region).
- **Mobile:** new UI must work at phone widths, and touch handlers should use `{ passive: false }` with `preventDefault()` where they need to stop scrolling or zoom.
- **No new globals.** Everything new goes inside the IIFE.

## Verifying changes

There is no automated test suite or linter. Before reporting work as done:

1. Check the script parses: `node -e "const s=require('fs').readFileSync('index.html','utf8');new Function(s.match(/<script>([\s\S]*)<\/script>/)[1])"` (this checks syntax only).
2. For gameplay changes, load the page in a headless browser (Playwright with the pre-installed Chromium at `/opt/pw-browsers/chromium`, if available) and confirm there are no console errors while starting a game, firing, and triggering the changed behavior. The most recent commit was verified this way.
3. Test at a phone-sized viewport if the change touches layout or touch controls.

If something cannot be run in this environment, say so explicitly rather than claiming it works.

## Git workflow

- Develop on the branch designated for the task (currently `claude/gifted-tesla-bk0uro`). Do not push to other branches without permission.
- Write commit messages in the imperative mood, with a short summary line and a body that explains the why for non-trivial changes. Follow the style of the existing commit `Add Sky Fighter: Raiden-style vertical shoot-em-up game`.
- Do not create pull requests unless explicitly asked.
