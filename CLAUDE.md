# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Google_4 is a small static web project containing **Sky Fighter**, a Raiden-style vertical shoot-'em-up built as a single-page HTML5 canvas game. There is no build step, no package manager, and no test suite.

## Repository layout

- `index.html` — the entire game: markup, CSS (`<style>`), and JavaScript (`<script>`) in one file (~1,670 lines).
- `README.md` — minimal project title only.
- `CLAUDE.md` — this file.

## Running the game

Open `index.html` directly in a browser, or serve the directory with any static server (for example `python3 -m http.server`). No install step is needed.

## Code structure (`index.html`)

- **Markup** (~lines 1–420): HUD (`#hud`, `#hearts`, `#bombs`, `#weaponStat`, `#multStat`), the `#gameCanvas` element, the `#overlay` start/pause/game-over card, mute/motion/pause buttons, and `#touchControls` (joystick, FIRE, BOMB).
- **Styles** (`<style>`, ~lines 10–420): HUD pills, overlay card, vignette/flash effects, and a `.reduced-motion` mode that disables screen shake, flashes, and pulses.
- **Script** (`<script>`, starts ~line 422): wrapped in a single IIFE `(function () { ... })();`. Sections are marked with `// ---------- Name ----------` comments.

Key parts of the script, roughly in order:

| Area | Notes |
| --- | --- |
| Config | `BASE_W = 480`, `BASE_H = 720` logical canvas size; `DIFFICULTIES`; `KILLS_PER_STAGE = 18`; `THEMES` (per-stage sky palettes); `ENEMY_TYPES`; `BOSSES` |
| Persistence | `localStorage` keys `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion` |
| Performance | `trackPerf` adjusts a `quality` level (0–2) from a rolling frame-time average |
| Audio | Lazily created `AudioContext` (`ensureAudio`, called from the Start button click); `beep`, `noiseBurst`, and the `sfx` table. |
| Resize | `resize()` scales the canvas to fit the window, capped at 1.6×, keeping the 480×720 logical resolution |
| Input | `keys` map from `keydown`/`keyup` using `e.code`; mouse, pointer, and touch joystick handlers |
| Game state | `resetGame()`, `update(dt)`, `draw()`, `loop(now)` driven by `requestAnimationFrame`; `dt` is clamped to 0.05 s |
| Weapons | `fireWeapon()` (Vulcan/Laser, 3 levels), `applyPowerUp()`, `useBomb()` |
| Bosses | `spawnBoss()`, `updateBoss()`, `killBoss()`; attack patterns are the `fire*` helpers (radial, aimed spread, columns, spiral, cross, homing, double ring) |
| Flow | `loseLife()`, `endGame()`, `togglePause()` |

## Controls

- Mouse: move to steer, click to fire.
- Keyboard: Arrows or WASD to move, Space to fire, X or Shift to bomb, P or Esc to pause.
- Touch: drag on the left side to fly with the virtual joystick, tap FIRE, tap BOMB.

## Conventions

- Keep everything in `index.html` unless there is a clear reason to split files. Do not add a build system, bundler, or npm dependencies without being asked.
- Match the existing style: 2-space indentation, single quotes in JS, `const`/`let` (no `var`), and the IIFE wrapper. Keep the section-header comments.
- Gameplay tuning lives in the config objects near the top of the script (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `KILLS_PER_STAGE`). Prefer changing those over adding magic numbers in update logic.
- New user-facing toggles should persist through a `skyfighter.*` `localStorage` key and respect the reduced-motion setting where they produce motion or flashing.
- Any new DOM lookup belongs with the existing `getElementById` block at the top of the script.
- Keep the game playable on phones: the layout and touch controls must keep working at small widths and in both orientations.

## Verification

There are no automated tests or linters. To verify a change:

1. Open `index.html` in a browser (or serve it locally) and check the browser console for errors.
2. Play through at least one stage, a boss fight, a bomb, a pause/resume, and a game over.
3. Check the mute, reduced-motion, and difficulty toggles, and that the best score persists after a reload.
4. For layout changes, check a narrow viewport as well as a desktop one.

## Git

- Develop on the branch the task specifies (currently `claude/gifted-tesla-0y47ya`). Do not push to other branches without permission.
- Write clear, descriptive commit messages that explain why the change was made.
