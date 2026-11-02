# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a small, dependency-free browser game, **Sky Fighter**: a Raiden-style vertical shoot-'em-up. The entire game lives in one file, `index.html` (~1,670 lines: inline CSS, markup, and a single inline `<script>` IIFE). There is no build step, no package manager, no test suite, and no server-side code.

Other files:
- `README.md` — placeholder title only.
- `.git` — history: `Initial commit`, then `Add Sky Fighter: ...`.

## Running the game

No build is needed. Open `index.html` directly in a browser, or serve the directory with any static server, for example:

```bash
python3 -m http.server 8000   # then visit http://localhost:8000/index.html
```

## Architecture of `index.html`

Layout, top to bottom:

1. **`<style>`** (around lines 10–362) — all styling, including the `body.reduced-motion` class and mobile/touch layout.
2. **Markup** (around lines 363–421) — `#gameCanvas`, the HUD (`#score`, `#best`, `#mult`, `#weaponName`, `#weaponLvl`, `#hearts`, `#bombs`), the overlay (`#overlay`, `#startBtn`, `#finalScore`, etc.), boss bar (`#bossBarWrap`), touch controls (`#touchControls`, `#moveZone`, `#fireBtn`, `#bombBtn`), and the `.diffBtn` buttons.
3. **`<script>`** (starts around line 422) — one IIFE with `'use strict'`. Everything is a closure-scoped `const`/`let`/`function`; nothing is exported to `window`.

Within the script, code is grouped by `// ----------` section comments and roughly follows this order:

| Area | What it holds |
| --- | --- |
| Constants | `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES` (per-stage sky palettes), `ENEMY_TYPES`, `BOSSES`, `STORAGE_*` localStorage keys |
| Accessibility / perf | `announce()` (aria-live), `trackPerf()` adaptive `quality` (0 full / 1 medium / 2 low) |
| Audio | `ensureAudio()`, `beep()`, `noiseBurst()`, `panFor()`; audio is created lazily on first user gesture |
| HUD / FX | `renderHearts`, `renderBombs`, `flashHit`, `showComboPop`, `showStageBanner`, `screenShake` |
| Input | Keyboard (`keydown`/`keyup` on `window`), mouse/pointer via `canvasPointFromEvent`, virtual joystick (`joystick*`), fire/bomb touch buttons |
| Game lifecycle | `resetGame()`, `update(dt)`, `endGame()`, `togglePause()`, `loseLife()`, `breakChain()` |
| Spawning & weapons | `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`, `applyPowerUp`, `fireWeapon`, `useBomb` |
| Boss AI | `BOSS_PATTERNS` (array of bullet-pattern functions such as `fireRadial`, `fireCross`, `fireHomingSeeded`), `updateBoss(dt)`, `killBoss()` |
| Rendering | `draw()`, `drawPlane`, `drawBossShip`, `drawClouds`, `resize()` (letterboxes to `BASE_W=480` × `BASE_H=720`) |
| Main loop | `loop(now)` — clamps `dt` to 0.05 s, then `update` + `draw` while `running && !paused`; started with `requestAnimationFrame(loop)` at the bottom |

Key global state is plain `let` variables inside the IIFE (score, lives, bombs, stage, boss, entity arrays, `running`, `paused`, `quality`, etc.). `resetGame()` is the place to reset them.

## Controls

| Input | Action |
| --- | --- |
| Mouse | Move to steer, click to fire |
| Arrows / WASD | Move |
| Space | Fire (hold) |
| X or Shift | Bomb |
| P or Esc | Pause / resume |
| Touch | Drag left zone to fly; FIRE and BOMB buttons |

Keep the `#controlsHint` text (around line 417) in sync if you change any binding.

## Persistence

Stored in `localStorage` under these keys (defined near the top of the script):

- `skyfighter.bestScore`
- `skyfighter.muted`
- `skyfighter.difficulty`
- `skyfighter.reducedMotion`

All reads are unguarded `localStorage.getItem` calls, so if you add storage access, consider wrapping it in try/catch for browsers with storage disabled.

## Conventions

- **Single file.** Keep the game self-contained in `index.html` unless the user asks for a split. Do not add npm dependencies or a build step without being asked.
- **Vanilla JS and Canvas 2D.** No frameworks. Use `requestAnimationFrame` timing via `dt` (seconds); scale movement by `dt` rather than per frame.
- **Tuning lives in constants.** Difficulty, enemy stats, boss HP multipliers, and theme colors are data objects at the top of the script; prefer editing those over hard-coding numbers in logic.
- **Respect the reduced-motion and quality paths.** New particle or shake effects should check `reducedMotion` and the `quality` level.
- **Accessibility.** Announce significant events through `announce()` and keep the ARIA live region working.
- **Keep the mobile layout working.** Test both desktop and touch-sized viewports when changing HUD or controls.
- **Comments:** section headers use `// ---------- Name ----------`. Match that style.

## Verifying changes

There are no automated tests or linters configured. To verify a change:

1. Open `index.html` in a browser (or serve it statically) and check the browser console for errors.
2. Start a run, exercise the changed system, and confirm pause (P/Esc), bomb (X/Shift), game over, and restart still work.
3. For touch changes, use the browser's device emulation.

If you add a test harness or tooling, document the command here.

## Git workflow

- Develop on the branch specified by the task (currently `claude/gifted-tesla-mtx0sf`); do not push to other branches without permission.
- Write clear, descriptive commit messages.
- Do not open pull requests unless explicitly asked.
