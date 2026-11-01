# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Google_4 is a small static web project. Its only app is **Sky Fighter**, a Raiden-style
vertical shoot-'em-up that runs as a single self-contained HTML5 canvas page.

- `index.html` (~1,670 lines): the entire game — markup, CSS, and JavaScript, all inline.
- `README.md`: placeholder title only.

There is no build step, no package manager, no bundler, no test suite, and no external
dependencies. The game runs by opening `index.html` in a browser.

## Repository layout

```
.
├── index.html   # Complete game: <style>, HUD/overlay markup, one IIFE <script>
├── README.md    # Minimal readme
└── CLAUDE.md    # This file
```

## Running and verifying changes

- Open `index.html` directly in a browser, or serve the folder with any static server
  (e.g. `python3 -m http.server` and visit `/index.html`).
- There are no automated tests or linters configured. Verify changes by playing the game
  in a browser, including the affected code path (see "Testing checklist" below).

## Architecture of `index.html`

**Markup (top of file).** A `#container` holds `#gameCanvas`, a damage `#vignette`, a
`#flash` overlay, the `#hud` (score, best, multiplier, weapon, hearts, bombs), the boss bar
(`#bossBarWrap`), combo and stage popups, the top-right icon buttons (`#muteBtn`,
`#motionBtn`, `#pauseBtn`), touch controls (`#touchControls` with joystick, FIRE, and BOMB),
and the start/game-over `#overlay` with difficulty buttons (`.diffBtn`).

**Script.** All logic lives in one `(function () { 'use strict'; ... })()` IIFE. Nothing is
exposed on `window`. Key sections, in file order:

| Area | Notes |
| --- | --- |
| Persistence | `localStorage` keys: `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion`. Every access is in the IIFE; keep new keys under the `skyfighter.` prefix. |
| Config tables | `DIFFICULTIES` (spawn timing, enemy/bullet tuning per level), `ENEMY_TYPES` (each has `minStage`), `BOSSES` (patterns, cycled by stage), `THEMES` (per-stage palette, cycled by stage), `KILLS_PER_STAGE`. |
| Audio | Web Audio API, created lazily on first user gesture via `ensureAudio()`. `beep()` and `noiseBurst()` produce all sound effects (`sfx` object). |
| Input | Keyboard (arrows/WASD move, Space or hold to fire, X or Shift bomb, P/Esc pause), mouse (steer and click to fire), touch (joystick, FIRE, BOMB). `keys[]` map plus `firing` flag. |
| Game state | Module-level `let`s: score, lives, bombs, weapon and level, multiplier, `stage`, `stageKills`, `frame`, plus arrays of entities. `resetGame()` initializes them. |
| Spawning | `spawnEnemy()`, `spawnBoss()`, `maybeSpawnPowerUp()`. Spawn interval ramps with `frame`. |
| Combat | `fireWeapon()` (Vulcan and Laser, 3 levels), `useBomb()`, `addScore()` with kill-chain multiplier, `breakChain()`. Boss attacks are the `fire*` pattern functions (radial, aimed spread, columns, spiral, cross, homing, double ring) driven by `updateBoss()`. |
| Update/draw | `update(dt)` advances the simulation with a delta-time step; `draw()` renders everything, including `drawPlane`, `drawBossShip`, and parallax `drawClouds`. `loop(now)` runs both via `requestAnimationFrame`. |
| Flow | `loseLife()`, `killBoss()` (advances stage), `endGame()`, `togglePause()`, `setDifficulty()`. |
| UI feedback | `showComboPop`, `showStageBanner`, `screenShake`, `flashHit`/`flashBomb`, `renderHearts`/`renderBombs`, and `announce()` for an `aria-live` region. |
| Accessibility | Reduced-motion toggle disables shake, flash, and pulse. `#srLive` announces stage changes, boss defeat, and game over. |
| Perf | `trackPerf(dt)` monitors frame time. |

## Conventions

- **Keep everything in `index.html`** unless the user asks for a split. The single-file
  design is intentional: it is meant to open directly from disk with no build.
- **Vanilla JS only.** Do not add frameworks, bundlers, or CDN dependencies without asking.
- **Preserve the IIFE and `'use strict'`.** New state should be a `let` inside the IIFE.
- **Use the existing config tables** (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `THEMES`)
  for tunable values instead of adding magic numbers in update logic.
- **Use `dt`-based movement** in `update()` so speed is frame-rate independent.
- **Respect the reduced-motion setting** when adding any new shake, flash, or pulse effect.
- **Make sounds through `beep()`/`noiseBurst()`** and check `muted` first; do not create
  new `AudioContext`s.
- **Keep the mobile layout working.** Touch controls, `touch-action: none`, and the
  `resize()` handler are all in play; check portrait and landscape when changing HUD or
  canvas sizing.
- **Match the existing style:** 2-space indentation, single quotes in JS, arrow functions for
  callbacks, `const` by default.

## Testing checklist (manual)

When you change gameplay or UI, exercise the relevant items in a browser:

1. Start, play, and die: game over shows score, stage, and best; best persists after reload.
2. Pause (P / Esc / pause button) and resume; the game does not advance while paused.
3. Each difficulty button works and persists across reload.
4. Mute and reduced-motion toggles persist across reload.
5. Weapon power-ups upgrade Vulcan/Laser levels; bombs clear enemies and damage bosses.
6. Clear a stage's kills to trigger the boss, defeat it, and confirm the next stage starts
   with the next theme and the stage banner shows.
7. Touch layout: joystick, FIRE, and BOMB work; the HUD does not overlap the controls.

## Git and workflow

- Default branch is `main`. Feature work goes on the branch named for the task
  (currently `claude/gifted-tesla-jdt578`).
- Write clear commit messages that describe the gameplay or UI change.
- Do not open pull requests unless explicitly asked.
