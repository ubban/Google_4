# CLAUDE.md

Guidance for Claude Code when working in this repository.

## What this repo is

`Google_4` is a single-page HTML5 canvas game called **Sky Fighter**, a Raiden-style vertical shoot-em-up. The whole game lives in one file, `index.html` (about 1,670 lines): inline CSS in `<style>`, markup in `<body>`, and one IIFE-wrapped `<script>` containing all game logic. `README.md` is a placeholder.

There is no build step, package manifest, test suite, linter, or CI config. To run the game, open `index.html` in a browser, or serve the directory with any static server.

## Layout of `index.html`

| Section | Approx. lines | Contents |
| --- | --- | --- |
| `<style>` | 10–360 | HUD pills, hearts/bombs, glassmorphic overlay card, touch controls, `.reduced-motion` and mobile rules |
| `<body>` | 361–421 | `#gameCanvas`, HUD elements, overlay (title/difficulty/start), touch controls, `#srLive` live region |
| Config constants | 463–527 | `STORAGE_*` keys, `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES`, `ENEMY_TYPES`, `BOSSES` |
| Audio | 543–678 | Lazy `AudioContext` (`ensureAudio`), `beep`, `noiseBurst`, `EXPLODE_SFX` |
| UI helpers | 680–752 | `renderHearts`, `renderBombs`, `flashHit`, `showComboPop`, `showStageBanner`, `screenShake`, `setDifficulty` |
| Layout and input | 753–870 | `resize` (fixed `BASE_W × BASE_H` = 480×720, scaled to fit), joystick, mouse/touch point mapping, keyboard handlers |
| State and reset | 865–912 | Top-level `let` game state, `resetGame` |
| Drawing | 913–1021, 1507–1639 | `drawPlane`, `drawBossShip`, `drawClouds`, `draw` |
| Spawning and combat | 1022–1147 | `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`, `spawnExplosion`, `addScore`/`breakChain`, `useBomb`, `fireWeapon` |
| Simulation | 1148–1303 | `update(dt)`: player movement, firing, collisions, enemy/bullet/power-up updates |
| Power-ups and bosses | 1304–1506 | `applyPowerUp`, boss bullet patterns (`BOSS_PATTERNS`), `updateBoss`, `killBoss`, `loseLife`, `endGame`, `togglePause` |
| Main loop | 1640–1668 | `loop(now)` via `requestAnimationFrame`, `dt` clamped to 50 ms; `startBtn` handler |

Line numbers are approximate and will drift as the file changes. Search by function or constant name rather than trusting them.

## Architecture notes

- **Single global-state IIFE.** Everything is scoped inside `(function () { 'use strict'; … })()`. Do not introduce top-level globals. Keep new code inside that scope.
- **Fixed logical resolution.** Gameplay coordinates use `BASE_W = 480`, `BASE_H = 720`. `resize()` handles the canvas scale. Positions, sizes and speeds are in logical pixels, not CSS pixels.
- **Frame-based timing.** `update(dt)` takes seconds, and `dt` is clamped to 0.05 s. Speeds should be multiplied by `dt`. Timers are decremented in seconds.
- **Adaptive quality.** `trackPerf` samples the last 45 frame times and steps `quality` between 0 (full), 1 (medium) and 2 (low). It currently scales explosion particle counts, shadow glow and cloud/glow passes. Respect `quality` when adding expensive effects.
- **Data-driven content.** Enemy types, bosses, themes and difficulty presets are plain object tables near the top of the script. Add new content there and reference it by key, rather than hard-coding values in the logic.
- **Boss patterns.** `BOSS_PATTERNS` is an array of functions that fire enemy bullets (radial, aimed spread, columns, spiral, cross, homing, double ring). Bosses cycle through these by phase.
- **Persistence.** The only persisted state is in `localStorage`, under keys prefixed `skyfighter.` (`bestScore`, `muted`, `difficulty`, `reducedMotion`). Every access is a plain `localStorage` call with no try/catch, so keep new writes consistent with that unless you are deliberately changing the pattern.
- **Audio is lazy.** `ensureAudio()` creates the `AudioContext` on the first user gesture. Do not create audio nodes before that point. Respect `muted`.
- **Accessibility.** `reducedMotion` (from the OS preference or the stored override) toggles the `reduced-motion` body class. `screenShake` scales magnitude and duration down when it is on, rather than disabling shake outright. Announcements to screen readers go through `announce()`, which writes to `#srLive`.
- **Input.** Keyboard: arrows/WASD move, Space fires, X or Shift bombs, P or Esc pauses. Mouse steers and clicks to fire. Touch uses a left-side joystick zone plus FIRE and BOMB buttons. Any new control should be added to the handlers and to the `#controlsHint` text.

## Working in this repo

- **Verify in a browser.** There are no automated tests. For any gameplay or rendering change, open `index.html` in a browser and play through the relevant path. Check the desktop and touch layouts when UI changes, and both light and reduced-motion modes when visuals change.
- **Keep it a single file.** Unless the user asks otherwise, add code to `index.html` rather than splitting out new modules, build tooling or dependencies. The project is deliberately dependency-free.
- **Match the existing style.** Use 2-space indentation, single quotes in JS, `const`/`let`, arrow functions for callbacks, and section banner comments of the form `// ---------- Name ----------`. Keep comments brief and explain *why*, not *what*.
- **Don't break the single-file contract.** Do not add external script or stylesheet dependencies, CDN links, or build outputs.
- **Commits.** Use short, imperative commit messages that describe the gameplay or UI change, as the existing history does (`Add Sky Fighter: …`).
- **Branches.** Develop on the branch named in the task instructions. Do not push to other branches without explicit permission.
