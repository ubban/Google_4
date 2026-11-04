# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a small static web project. Its only real content is **Sky Fighter**, a
Raiden-style vertical shoot-'em-up written as a single-page HTML5 canvas app.

| File         | Purpose                                                        |
|--------------|----------------------------------------------------------------|
| `index.html` | The entire game: markup, CSS (`<style>`), and JS (`<script>`). |
| `README.md`  | Placeholder README (project title only).                       |
| `CLAUDE.md`  | This file.                                                     |

There is no build step, no package manager, no bundler, no test suite, and no
server-side code. The game runs by opening `index.html` in a browser.

## Running and checking changes

- Open `index.html` directly in a browser, or serve the directory with any static
  server (e.g. `python3 -m http.server` from the repo root) and visit it.
- There are no automated tests or linters configured. Verify changes by playing the
  game in a browser: start a run, fire, use a bomb, pause/resume, lose a life, reach a
  stage boss, and game over.
- For UI or layout changes, check both a desktop viewport and a phone-width viewport
  (the touch controls only appear on touch devices).

## Architecture of `index.html`

The game is one IIFE (`(function () { 'use strict'; ... })();`) inside a single
`<script>` tag. Keep new code inside that scope rather than adding globals.

Major sections, in file order (search for the `// ----------` comment banners):

1. **DOM references and constants**: cached element lookups, `STORAGE_*` keys,
   `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES`, `ENEMY_TYPES`, `BOSSES`.
2. **Settings and persistence**: best score, mute, difficulty, and reduced-motion are
   stored in `localStorage` under the `skyfighter.*` keys. Every `localStorage` access
   should stay defensive about missing values.
3. **Audio**: a lazily created Web Audio context (`ensureAudio`), with `beep` and
   `noiseBurst` helpers. Audio starts only after a user gesture.
4. **HUD and feedback**: hearts, bombs, combo pops, stage banner, screen shake, hit
   flash, and screen-reader announcements via `announce()`.
5. **Layout and input**: `resize()` scales the fixed `BASE_W × BASE_H` (480 × 720)
   playfield to the viewport. Keyboard handlers, a virtual joystick, and fire/bomb
   touch buttons feed a `keys` map and the `firing` flag.
6. **Game state**: `resetGame()` initializes state; `update(dt)` advances the simulation.
7. **Weapons and bombs**: `fireWeapon()` (Vulcan/Laser, 3 power levels), `useBomb()`,
   and `applyPowerUp()`.
8. **Enemies and bosses**: `spawnEnemy()`, `spawnBoss()`, `BOSS_PATTERNS` (radial,
   aimed spread, columns, spiral, cross, homing, double ring), `updateBoss()`,
   `killBoss()`.
9. **Rendering**: `draw()` renders the parallax clouds, planes (`drawPlane`), boss ships
   (`drawBossShip`), bullets, power-ups, and particles on the canvas.
10. **Loop and lifecycle**: `loop(now)` runs on `requestAnimationFrame` with `dt` capped
    at 0.05 s. `loseLife()`, `endGame()`, and `togglePause()` handle run flow. An adaptive
    `quality` level (full / medium / low) is adjusted by `trackPerf()` from frame times.

## Controls (keep docs and code in sync)

- Move: mouse (steer by pointer), arrow keys, or WASD. Touch: drag the joystick on the left.
- Fire: click, or hold Space. Touch: hold the fire button.
- Bomb: X or either Shift key. Touch: bomb button.
- Pause: P or Escape.
- HUD buttons `muteBtn`, `motionBtn` (reduced motion), and `pauseBtn`, plus the
  `.diffBtn` difficulty buttons on the start overlay.

The in-game hint text is the `#controlsHint` element. If you change a control, update
this section and `#controlsHint` together.

## Conventions

- **Single file.** Keep HTML, CSS, and JS in `index.html` unless the user asks for a
  split. Do not add a build step or dependencies without asking.
- **No external network dependencies.** Keep the game working offline. Don't add CDN
  scripts or fonts without an explicit request.
- **Vanilla JS and canvas only.** Match the existing style: `const`/`let`, arrow
  functions, two-space indentation, single quotes, and section banners.
- **Time-based updates.** Movement and timers use `dt` (seconds). Do not tie logic to
  frame count.
- **Respect reduced motion.** Any new screen shake, flash, or particle effect should
  check the `reducedMotion` setting.
- **Accessibility.** Announce important state changes through `announce()`, and keep
  new interactive controls keyboard- and touch-reachable.
- **Mobile first.** The viewport meta disables user scaling. Test touch behavior and keep
  controls out of the way of the playfield.

## Tuning

Gameplay numbers live in named constants near the top of the script:
`DIFFICULTIES` (spawn rate, enemy speed, shot chance, lives, bombs, boss HP multiplier),
`KILLS_PER_STAGE`, `ENEMY_TYPES`, and `BOSSES`. Prefer changing these constants over
hard-coding new magic numbers in update logic.

## Git workflow

- Develop on the branch the session specifies (currently `claude/gifted-tesla-o500cx`).
  Do not push to other branches without permission.
- Write clear commit messages that describe the gameplay or code change.
- Do not open a pull request unless the user asks for one.
