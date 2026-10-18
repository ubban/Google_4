# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a small, dependency-free web game called **Sky Fighter**: a Raiden-style vertical shoot-'em-up that runs in a single HTML page on a 2D canvas.

The repository is intentionally minimal:

| Path        | Purpose                                                        |
|-------------|----------------------------------------------------------------|
| `index.html`| The entire game: markup, CSS, and JavaScript in one file (~1,670 lines). |
| `README.md` | Placeholder title only.                                        |

There is no build step, no package manager, no test suite, and no linter configuration.

## Running the game

Open `index.html` directly in a browser, or serve the directory with any static server, for example:

```sh
python3 -m http.server 8000   # then visit http://localhost:8000
```

## Code layout inside `index.html`

The file is organized in four sections, in this order:

1. **`<style>` (around lines 10–359)**: all CSS, including the HUD, overlay, touch controls, boss bar, and a `body.reduced-motion` override block.
2. **`<body>` (around lines 361–421)**: static DOM. Key IDs the script depends on: `gameCanvas`, `overlay`, `startBtn`, `hearts`, `bombs`, `muteBtn`, `motionBtn`, `pauseBtn`, `touchControls`, `moveZone`, `joystickBase`, `fireBtn`, `bombBtn`, `bossBarWrap`, `srLive`.
3. **`<script>` (around lines 422–1667)**: one IIFE with `'use strict'`. Nothing is exported to `window`.
4. Closing tags.

Inside the script, code is grouped by `// ---------- Section ----------` comments. Keep that convention when adding code. The main sections are:

- **Config tables**: `DIFFICULTIES`, `THEMES` (six stage backdrops that cycle), `ENEMY_TYPES` (`grunt`, `interceptor`, `tank`), `BOSSES` (six boss designs that cycle by stage), `KILLS_PER_STAGE` (18).
- **Persistence**: `STORAGE_*` keys (`skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion`) read and written through `localStorage`.
- **Audio**: a lazily created `AudioContext` (`ensureAudio`), a `beep` / `noiseBurst` synth, and an `sfx` object of named sound effects. Audio starts only after a user gesture.
- **Canvas sizing**: fixed logical resolution `BASE_W × BASE_H` (480 × 720), scaled to the window with a cap of 1.6×.
- **Input**: keyboard (`keys` map keyed by `e.code`), mouse/pointer aim, and a touch joystick plus FIRE/BOMB buttons.
- **Game state**: `resetGame()` initializes `player`, `bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`, `boss`, and counters such as `score`, `lives`, `stage`, `kills`.
- **Entities and spawning**: `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp` (power-up types: `life`, `bomb`, `weapon`), `spawnExplosion`.
- **Weapons and bombs**: `fireWeapon` (`vulcan` and `laser`, levels 1+), `useBomb`, `applyPowerUp`.
- **Boss attacks**: pattern functions (`fireRadial`, `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`, `fireCross`, `fireHomingSeeded`, `fireDoubleRing`) listed in `BOSS_PATTERNS`, driven by `updateBoss`.
- **Game flow**: `update(dt)`, `draw()`, `loseLife`, `killBoss`, `endGame`, `togglePause`.
- **Main loop**: `loop(now)` runs on `requestAnimationFrame`, clamps `dt` to 50 ms, and calls `update` and `draw` only while running and not paused.
- **Adaptive quality**: `trackPerf` samples the last 45 frame times and steps a `quality` level (0–2) up or down to hold roughly 40–56 FPS.

## Game rules and tuning

- **Difficulty** (`easy` / `normal` / `hard`) sets spawn cadence, enemy speed, enemy fire chance, starting lives, starting bombs, and boss HP multiplier. The selection persists across visits.
- **Stages**: once `stageKills` reaches `KILLS_PER_STAGE` (18), a single boss is spawned for that stage. Stage number selects the boss (`BOSSES[(stage - 1) % BOSSES.length]`) and the backdrop theme (`THEMES[(stage - 1) % THEMES.length]`).
- **Enemies**: `grunt` (1 HP, stage 1+), `interceptor` (weaving, fast, stage 1+), `tank` (4 HP, stage 2+).
- **Lives and bombs** come from the difficulty table and are refilled by pickups.
- **Scoring**: base points per enemy type, multiplied by a combo chain that `breakChain` resets when the player is hit.

When changing balance, edit the constant tables rather than the logic that reads them.

## Conventions

- **Single file, no dependencies.** Do not introduce a bundler, framework, or external script/CSS dependency without an explicit request. The game is meant to open directly from disk.
- **Match the existing style.** The file uses 2-space indentation, single quotes in JavaScript, `const` by default, arrow callbacks, and short helper functions. Section headers use the `// ---------- Name ----------` form.
- **Accessibility is part of the feature set.** Preserve the `srLive` announcements, `aria-*` attributes, the `sr-only` helper, the reduced-motion toggle (`body.reduced-motion`), and `prefers-reduced-motion` handling when touching related code.
- **Mobile is supported.** Any new control needs a touch path as well as keyboard and mouse. Keep `touch-action` and `preventDefault` handling consistent with the existing touch listeners.
- **Keep the game loop allocation-light.** Hot paths (`update`, `draw`, and the boss pattern functions) run every frame; avoid creating new arrays or objects in them where a reused value would do.
- **Persist only through the existing `STORAGE_*` keys.** If you add a new persisted setting, follow the same pattern and wrap access so the game still works when storage is unavailable.

## Verifying changes

There are no automated tests or lint scripts. To verify a change:

1. Open `index.html` in a browser, or serve it locally.
2. Exercise the affected path: start a game, switch difficulty, collect power-ups, trigger a bomb, reach a boss, pause/resume, and reach game over.
3. Check the browser console for errors. The script runs in strict mode, so most mistakes surface as exceptions.
4. For input changes, test keyboard, mouse, and (in device emulation) touch.
5. Confirm that the best score and settings survive a page reload.

If you add tests or tooling later, update this section with the exact commands.

## Git workflow

- Develop on the branch specified by the session or task; do not push to other branches without permission.
- Write descriptive commit messages that explain *why* a change was made.
- Do not open pull requests unless explicitly asked.
