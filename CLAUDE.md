# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo contains **Sky Fighter**, a single-file, browser-based vertical shoot-em-up (Raiden-style) game. There is no build step, package manager, framework, or test suite.

- `index.html` (~1,670 lines): the entire game. HTML markup at the top, CSS in one `<style>` block, and vanilla JavaScript in one `<script>` block wrapped in an IIFE (`(function () { 'use strict'; ... })();`).
- `README.md`: a placeholder title. It contains no setup instructions.

## Running the game

No build or install is needed. Open `index.html` directly in a browser, or serve the directory with any static server, for example:

```bash
python3 -m http.server 8000   # then visit http://localhost:8000/index.html
```

## Repository layout

```
index.html   # The whole game: markup, styles, and script
README.md    # Project title only
CLAUDE.md    # This file
```

Keep the game in `index.html` unless there is a clear reason to split it. The project is intentionally dependency-free.

## Code structure inside `index.html`

The script is organized into sections, marked with `// ---------- Name ----------` comments. Follow that convention when adding code.

| Area | What it covers |
|---|---|
| DOM lookups | `getElementById` references at the top of the IIFE (HUD, overlay, touch controls, buttons) |
| Storage keys | `STORAGE_BEST`, `STORAGE_MUTE`, `STORAGE_DIFF`, `STORAGE_REDUCED` (all prefixed `skyfighter.`) |
| Adaptive quality | `trackPerf` lowers or raises a `quality` level (0 full, 1 medium, 2 low) based on a 45-frame average |
| Config tables | `DIFFICULTIES`, `THEMES` (stage backdrops), `ENEMY_TYPES`, `BOSSES`, `BOSS_PATTERNS` |
| Audio | Web Audio helpers `ensureAudio`, `beep`, `noiseBurst`, and the `sfx` object. Audio is created lazily on first user gesture. |
| Input | Keyboard (`keys`), touch joystick and buttons, and mouse steering on the canvas |
| Game state | `resetGame()` initializes state. `update(dt)` runs the simulation and `draw()` renders it. |
| Combat | `spawnEnemy`, `spawnBoss`, `fireWeapon`, `useBomb`, `addScore`, combo/chain handling, `applyPowerUp` |
| Boss patterns | `fireRadial`, `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`, `fireCross`, `fireHomingSeeded`, `fireDoubleRing`, used by `updateBoss` |
| Lifecycle | `loseLife`, `killBoss`, `endGame`, `togglePause` |
| Rendering | `drawPlane`, `drawBossShip`, `drawClouds`, `draw()`, plus the canvas `resize()` that scales a 480×720 logical space |
| Loop | `loop(now)` caps `dt` at 0.05s, then calls `trackPerf`, `update`, and `draw` while running and not paused |

### Game constants

- Logical resolution: `BASE_W = 480`, `BASE_H = 720`.
- `KILLS_PER_STAGE = 18`. Stage progression advances after this many kills.
- Difficulty (`easy` / `normal` / `hard`) controls spawn rate, enemy speed, enemy fire chance, starting lives and bombs, and boss HP multiplier.

## Persistence

`localStorage` keys, all under the `skyfighter.` prefix:

- `skyfighter.bestScore`: high score (written in `endGame`).
- `skyfighter.muted`: `'1'` when muted.
- `skyfighter.difficulty`: last selected difficulty name.
- `skyfighter.reducedMotion`: `'1'` or `'0'`. If unset, the `prefers-reduced-motion` media query decides the default.

Reads and writes to `localStorage` are not wrapped in try/catch. If you change persistence, consider that storage may be unavailable (private windows, blocked site data) and handle that case.

## Accessibility and UX conventions

- Announce important state changes through `announce(text)`, which writes to the `#srLive` live region.
- Reduced motion (`body.reduced-motion`) disables screen shake, the flash overlay, and pulses. Respect this for any new visual effect.
- The game supports keyboard, mouse, and touch. Any new control should work on all three, and touch buttons call `e.preventDefault()` with `{ passive: false }`.
- Interactive elements need visible labels or `title` attributes, as the existing icon buttons do.

## Working conventions

- **No dependencies.** Do not add npm packages, CDN scripts, or build tooling without a clear request. Keep changes vanilla JS and CSS.
- **Match the surrounding style.** The code uses 2-space indentation, single quotes in JS, `const` by default, arrow functions for callbacks, and short comments that label sections.
- **Keep the game loop cheap.** `update` and `draw` run every frame. Avoid unnecessary allocations in hot paths, and gate expensive effects on the `quality` level so the adaptive quality system can still work.
- **Time-based movement.** Movement and timers use `dt` (seconds). Do not tie speeds to frame counts.
- **Test manually.** There is no automated test suite. For gameplay or rendering changes, load `index.html` in a browser and check the affected flow: start, fire, bomb, pause, stage change, boss fight, game over, and best-score persistence. Check at least one mobile-width viewport for layout changes.

## Git

- Default development branch for this task: `claude/gifted-tesla-jt7vlz`.
- Commit with clear, descriptive messages. Push with `git push -u origin <branch>`.
- Do not open a pull request unless the user asks for one.
