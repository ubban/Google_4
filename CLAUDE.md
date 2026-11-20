# CLAUDE.md

Guidance for Claude Code when working in this repository.

## What this repo is

`Google_4` is a small static project. Its main content is **Sky Fighter**, a
Raiden-style vertical shoot-em-up written as a single-page HTML5 canvas app.

| File        | Purpose                                                        |
|-------------|----------------------------------------------------------------|
| `index.html` | The entire game: markup, CSS in a `<style>` block, and game logic in one inline `<script>` IIFE. |
| `README.md`  | Minimal project readme.                                        |

There is no build step, no package manager, no bundler, no test suite, and no
external dependencies. The game runs by opening `index.html` in a browser.

## Working on the game

- **Keep it a single file.** Styles, markup, and script all live in `index.html`.
  Do not split it into modules or add a build tool unless the user asks.
- **No dependencies.** Use plain browser APIs (canvas 2D, Web Audio, `localStorage`,
  `matchMedia`). Do not add CDN libraries or npm packages without asking.
- **Everything is inside an IIFE** with `'use strict'`. Do not add globals; expose
  nothing on `window` unless there is a clear reason.
- **Fixed logical resolution:** `BASE_W = 480`, `BASE_H = 720`. The canvas is
  always 480×720 internally and CSS-scaled to fit the window (max 1.6×). All
  positions, sizes, and speeds are in these logical pixels.

### Code map (inside the `<script>`)

- **Config tables:** `DIFFICULTIES` (spawn rate, lives, bombs, enemy fire chance,
  boss HP multiplier), `THEMES` (per-stage sky palettes), `ENEMY_TYPES`
  (`grunt`, `interceptor`, `tank`), `BOSSES` (six boss designs), `KILLS_PER_STAGE`.
- **Persistence keys:** `skyfighter.bestScore`, `skyfighter.muted`,
  `skyfighter.difficulty`, `skyfighter.reducedMotion`. Keep these names stable so
  saved player data survives changes.
- **Adaptive quality:** `trackPerf` / `quality` (0 full, 1 medium, 2 low) lowers
  effects when frame time is high.
- **Audio:** `ensureAudio`, `beep`, `noiseBurst`. Audio is created lazily on the
  first user gesture (browser autoplay rules). Respect `muted`.
- **Input:** keyboard (`keys` map), mouse on canvas, and touch controls
  (joystick in `moveZone`, `fireBtn`, `bombBtn`). Any new control needs both
  keyboard and touch paths.
- **Simulation:** `update(dt)` is the main loop step. `resetGame()` resets state.
  `fireWeapon()`, `useBomb()`, `applyPowerUp()`, `spawnEnemy()`, `spawnBoss()`,
  and `maybeSpawnPowerUp()` handle gameplay. Damage to the player is handled
  near `lives--`.
- **Rendering:** `drawPlane`, `drawBossShip`, plus HUD helpers `renderHearts`,
  `renderBombs`, `showComboPop`, `showStageBanner`, `screenShake`.

### Accessibility and UX conventions

- Respect `prefers-reduced-motion` (see `reducedMotion`). Any new screen shake,
  flash, or pulsing effect must be gated by it.
- Announce important state changes through `announce(text)`, which feeds the
  `aria-live` region (`#srLive`).
- Pause (`paused`) must stop the simulation. Check it when adding timers or
  time-based effects.

## Making changes

- Read the relevant section of `index.html` before editing. Line numbers shift,
  so search by identifier rather than by line.
- Keep edits small and local. Match the existing style: 2-space indent, single
  quotes in JS, `const` by default, short section comments like
  `// ---------- Section ----------`.
- When changing gameplay numbers, change them in the config tables rather than
  scattering literals through the code.
- Keep the touch and keyboard paths in sync.
- Test by opening `index.html` in a browser (or serving the directory with any
  static server) and playing through: start, fire, bomb, pause, game over, and
  a difficulty change. There are no automated tests, so say so in your summary if
  you could not play-test a change.

## Git

- Development branch for this repo: `claude/gifted-tesla-yokbpz`. Commit and push
  there. Do not push to other branches without explicit permission.
- Do not open a pull request unless the user asks for one.
- Commit messages: a short imperative subject, then a body explaining the why.
  Do not include secrets or personal contact details.

## Keeping this file current

Update this CLAUDE.md when the structure changes in a way that would mislead a
future session: new files, a split of `index.html`, a build or test setup, new
persistence keys, or new config tables. Do not record transient details such as
exact line numbers or the current high score.
