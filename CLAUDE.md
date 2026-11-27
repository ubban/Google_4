# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Google_4 is a small, dependency-free browser game: **Sky Fighter**, a Raiden-style vertical shoot-'em-up. The entire game lives in one file, `index.html` (~1,670 lines), containing inline `<style>`, markup, and one inline `<script>` IIFE. There is no build step, no package manager, no test suite, and no server requirement.

Files:
- `index.html` — the complete game (HTML, CSS, and JavaScript).
- `README.md` — minimal repo title.
- `CLAUDE.md` — this file.

## Running the game

- Open `index.html` directly in a browser, or serve the directory with any static server (for example `python3 -m http.server`).
- Verification is manual: play the game in a browser. For automated checks, Playwright is available in the cloud environment (use the pre-installed Chromium; do not run `playwright install`).

## Code layout inside `index.html`

- **Markup** (`<body>`): HUD (`#hud`, score, best, multiplier, weapon, hearts, bombs), boss bar (`#bossBarWrap`), overlay/menu (`#overlay`, start button, difficulty buttons `.diffBtn`, stats row), touch controls (`#touchControls`, `#moveZone`, `#fireBtn`, `#bombBtn`), and the `<canvas id="gameCanvas">`.
- **CSS** (`<style>`): all styling; colors and layout for HUD, overlay, and touch controls. Includes `safe-area-inset` handling for notched devices.
- **Script** (`<script>`): one IIFE with `'use strict'`. Key sections, roughly in order:
  - DOM references (`const xxxEl = document.getElementById(...)`).
  - Storage keys: `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion`.
  - Adaptive performance (`trackPerf`): averages the last 45 frame times and adjusts a `quality` level (0–2) up or down.
  - Data tables: `DIFFICULTIES` (easy/normal/hard), `KILLS_PER_STAGE` (18), `THEMES` (per-stage sky palettes), `ENEMY_TYPES` (grunt, interceptor, tank), `BOSSES` (six bosses, cycled by stage).
  - Audio: a lazily created `AudioContext` with `beep` and `noiseBurst` helpers; `ensureAudio()` must run from a user gesture.
  - Effects and HUD helpers: hearts/bombs rendering, hit/bomb flashes, combo pop, stage banner, screen shake.
  - Input: keyboard (`keys[e.code]`), mouse (steer and click-to-fire on the canvas), and touch (virtual joystick, fire and bomb buttons).
  - Game state: `resetGame()` and module-level `let` state (`score`, `lives`, `running`, `paused`, `gameOver`, `player`, `bullets`, `enemies`, `enemyBullets`, `powerUps`, `boss`, etc.).
  - Simulation: `update(dt)` (player, firing, spawning, collisions, power-ups), `spawnEnemy`, `spawnBoss`, `updateBoss`, `killBoss`, `loseLife`, `useBomb`, `fireWeapon`.
  - Boss attack patterns: `BOSS_PATTERNS` indexed by boss design, using helpers such as `fireRadial`, `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`, `fireCross`, `fireHomingSeeded`, `fireDoubleRing`.
  - Rendering: `draw()` plus `drawPlane` and `drawBossShip`. Screen shake is applied during drawing.
  - Lifecycle: `togglePause`, `endGame` (writes best score), and the `loop(now)` requestAnimationFrame loop with `dt` clamped to 50 ms.

## Game rules worth knowing

- Controls: arrows/WASD move, Space fires, X or Shift uses a smart bomb, P or Esc pauses. Mouse steers and clicks to fire. Touch uses the left drag zone to fly and on-screen FIRE and BOMB buttons.
- Weapons: `vulcan` and `laser`, each with a level. Weapon power-ups are type `weapon`; `life` and `bomb` power-ups also drop.
- Difficulty changes spawn rate, enemy speed, enemy fire chance, starting lives and bombs, and boss HP multiplier.
- Each stage needs `KILLS_PER_STAGE` kills before its boss spawns. Stages cycle through `THEMES` and `BOSSES`.
- Best score persists in `localStorage`; all storage access should stay tolerant of it being unavailable.

## Conventions

- Keep the game as a single self-contained file unless the user asks for a restructure. Do not add a bundler, framework, or npm dependencies without asking.
- Keep the code in the existing style: `const`/`let`, plain functions, 2-space indentation, single quotes in JS, and section comments like `// ---------- Name ----------`.
- Tuning values live in the data tables (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `BOSS_PATTERNS`). Prefer changing those over hard-coding numbers in logic.
- Preserve accessibility hooks: the `srLive` live region (`announce()`), the reduced-motion toggle, and the mute toggle. Preserve touch support and `safe-area-inset` padding.
- Preserve the `(function () { ... })();` wrapper so no globals leak.
- Do not add analytics, network calls, or third-party assets.

## Making changes

- Read the relevant section of `index.html` before editing; it is large, so use search (`grep -n`) to locate functions.
- After changes, verify in a browser: start a game, check movement and firing, a bomb, a boss fight, pause/resume, game over, and best-score persistence. Test on a narrow (phone-sized) viewport as well as desktop.
- Commit with clear messages and push to the branch the session is assigned; do not push to other branches without permission.
