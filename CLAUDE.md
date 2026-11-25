# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Google_4 is a small static web project containing **Sky Fighter**, a Raiden-style vertical shoot-'em-up played in the browser on a `<canvas>`.

Files:

- `index.html` — the entire game: markup, CSS, and JavaScript in a single file (~1,670 lines, one inline `<script>`).
- `README.md` — placeholder title only.

There is no build system, package manager, bundler, test suite, or linter. Nothing needs to be installed.

## Running and testing

- Open `index.html` directly in a browser, or serve the directory with any static server (e.g. `python3 -m http.server` and visit `http://localhost:8000`).
- There are no automated tests. Verify changes by playing the game in a browser, ideally at both desktop and phone-sized viewports (touch controls are shown on touch devices).
- When changing gameplay, check at least: all three difficulties, a stage transition, a boss fight, bomb use, pause/resume, game over, and the best-score update.

## Code layout (inside `index.html`)

- **CSS** (top of file): dark theme via `color-scheme: dark`; HUD, overlay, boss bar, and touch-control styles. Keep the layout working at phone width with no horizontal scroll.
- **Markup**: `#container` holds `#gameCanvas`, the HUD (`#score`, `#best`, `#mult`, `#weaponName`, `#hearts`, `#bombs`), boss bar, `#topBar` (mute, reduced-motion, pause buttons), touch controls (`#moveZone`/joystick, `#fireBtn`, `#bombBtn`), and `#overlay` (title/menu/game-over card).
- **Script constants**:
  - `DIFFICULTIES` — per-difficulty tuning (`easy`, `normal`, `hard`): spawn rate, enemy speed, enemy shoot chance, starting lives, starting bombs, boss HP multiplier.
  - `KILLS_PER_STAGE` (18) — kills needed to advance a stage.
  - `THEMES`, `ENEMY_TYPES` (`grunt`, `interceptor`, `tank`), `BOSSES` — stage and enemy definitions.
  - `BASE_W = 480`, `BASE_H = 720` — logical canvas size, scaled by `resize()`.
- **Persistence**: `localStorage` keys `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion`. Reads and writes are not wrapped in try/catch; if you add more storage, guard it so the game still runs when storage is blocked.
- **Audio**: Web Audio API, created lazily by `ensureAudio()` after a user gesture. Sound effects come from `beep()` and `noiseBurst()`.
- **Performance**: `trackPerf()` adjusts a `quality` level (full / medium / low) based on frame time.
- **Game loop**: `update(dt)` advances state each frame; drawing helpers (`drawPlane`, `drawBossShip`, etc.) render it. `resetGame()` restores initial state.
- **Weapons and bosses**: `fireWeapon()` and `applyPowerUp()` handle weapons; the `fire*` pattern helpers (`fireRadial`, `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`, `fireCross`, `fireHomingSeeded`, `fireDoubleRing`) implement boss attack patterns.
- **Accessibility**: `announce()` writes to the `#srLive` live region; reduced-motion mode disables screen shake, flashes, and pulses.

## Controls (for reference when testing)

- Keyboard: arrow keys / WASD to move, `Space` to fire (held), `X` or `Shift` for bomb, `P` or `Escape` to pause.
- Mouse: move the pointer over the canvas; mouse-down on the canvas and the fire button fire.
- Touch: left-side joystick to move, FIRE and BOMB buttons on the right.

## Conventions

- Keep everything in `index.html` unless there is a clear reason to split it. Do not introduce a build step or external dependencies without asking.
- Keep the existing style: 2-space indentation, `const`/`let`, single quotes in JS, and named constants for tuning values in the `DIFFICULTIES` / `BOSSES` / `ENEMY_TYPES` tables rather than magic numbers.
- Preserve the `localStorage` key names; changing them would reset players' saved best score and settings.
- Preserve mobile behavior: `touch-action: none` on the game, and no horizontal page scroll.
- Don't add network requests, analytics, or third-party scripts.

## Git

- Development branch: `claude/gifted-tesla-u0kfex`. Commit and push there; do not push to other branches without permission.
- Don't open a pull request unless asked.
