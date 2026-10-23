# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a small static web project. Its only real content is **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs in the browser on an HTML5 `<canvas>`.

- `index.html` — the whole game: markup, CSS, and JavaScript in one file (~1,670 lines).
- `README.md` — a placeholder with a title and a timestamp line. It has no setup or usage instructions.

There is no build step, no package manager, no bundler, no framework, no test suite, and no linter configuration. Do not add any of these unless the user asks.

## Running the game

Open `index.html` directly in a browser, or serve the directory with any static server, for example:

```sh
python3 -m http.server 8000
```

Then visit `http://localhost:8000/index.html`.

## Repository layout

```
.
├── index.html   # Sky Fighter: <style> (~lines 10–362), markup (~363–421), <script> (~422–1668)
└── README.md
```

Inside `index.html`, the `<script>` block is a single IIFE with `'use strict'`. Nothing is exported to `window`. Keep new code inside that IIFE unless there is a reason not to.

## Architecture of the game script

The script is organized in sections, marked by `// ---------- Name ----------` comments. Keep new code in the matching section.

| Area | Key identifiers | Notes |
| --- | --- | --- |
| DOM refs | `canvas`, `overlay`, `startBtn`, `heartsEl`, ... | All `getElementById` lookups are at the top of the IIFE. |
| Persistence | `STORAGE_BEST`, `STORAGE_MUTE`, `STORAGE_DIFF`, `STORAGE_REDUCED` | Keys are namespaced `skyfighter.*` in `localStorage`. |
| Adaptive quality | `quality`, `trackPerf()` | Averages the last 45 frame times and steps quality 0 (full) → 2 (low). |
| Config tables | `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES`, `ENEMY_TYPES`, `BOSSES`, `BOSS_PATTERNS` | Tune gameplay here rather than hard-coding numbers in logic. |
| Audio | `ensureAudio()`, `beep()`, `noiseBurst()`, `sfx` | Web Audio, synthesized at runtime. No audio files. |
| Input | `keys`, joystick helpers, `fireBtn` / `bombBtn`, mouse on `canvas` | Keyboard, touch joystick, and mouse are all supported. |
| Game state | `resetGame()`, `update(dt)`, `endGame()`, `togglePause()` | `update` is the fixed-logic step; `draw` is rendering. |
| Entities | `spawnEnemy()`, `spawnBoss()`, `maybeSpawnPowerUp()`, `spawnExplosion()` | Enemies are plain objects in arrays. |
| Weapons | `fireWeapon()`, `applyPowerUp()`, `player.weapon` (`vulcan` / `laser`), `player.level` (1–3) | Weapon upgrades cycle level 1 → 3, then switch weapon. |
| Bosses | `BOSS_PATTERNS`, `updateBoss()`, `killBoss()` | Each boss cycles through attack patterns such as radial, spiral, cross, and homing. |
| Rendering | `draw()`, `drawPlane()`, `drawBossShip()`, `drawClouds()` | Canvas 2D, drawn at a fixed logical size. |
| Main loop | `loop(now)` | `requestAnimationFrame`. `dt` is clamped to 50 ms. |

### Conventions in this codebase

- **Fixed logical resolution.** The canvas uses `BASE_W = 480`, `BASE_H = 720`, and `resize()` scales it to the viewport. Write game coordinates in those units.
- **Frame-rate independence.** Movement and timers use `dt` in seconds. Keep that pattern; do not count frames.
- **Delta cap.** `dt` is clamped to `0.05` in `loop()`. This prevents large jumps after the tab is backgrounded.
- **Accessibility.** Announcements go through `announce()` into the `#srLive` live region. Respect the reduced-motion setting (`reducedMotion`, `body.reduced-motion`) for screen shake and flashes.
- **Persistence.** Save only through the `STORAGE_*` constants. Wrap new `localStorage` access in a try/catch if it could run in a private window or with blocked storage. The current code does not do this, so match the existing pattern only where it is safe.
- **Audio gating.** Browsers require a user gesture before audio starts. Call `ensureAudio()` from input handlers before playing sound.
- **Naming and style.** Two-space indentation, single quotes in JavaScript, `const` by default, arrow functions for callbacks, and short helper functions named with verbs (`spawnX`, `fireX`, `drawX`, `applyX`).

## Working in this repo

- **Make small, local edits.** The file is large and tightly coupled. Prefer targeted changes over restructuring, and do not split `index.html` into modules unless asked.
- **Verify in a browser.** There are no automated tests. After a gameplay or rendering change, load the page and play through the affected path. If you cannot run a browser here, say so and state what was verified by reading the code.
- **Check both input paths.** Any change to controls should be checked for keyboard, touch (`touchControls`), and mouse input.
- **Check difficulty and quality scaling.** New enemies, bullets, or effects can affect frame time. Consider the `quality` levels, and confirm the game still runs at `quality = 2`.
- **Mind the storage keys.** Renaming a `skyfighter.*` key resets saved best score, mute, difficulty, or motion preferences for existing players.

## Git

- The default branch is `main`. Development work for this session happens on `claude/gifted-tesla-s8csv4`.
- Commit messages should be short and imperative, for example `Add Sky Fighter: Raiden-style vertical shoot-em-up game`.
- Do not open a pull request unless the user asks for one.
