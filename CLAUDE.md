# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Google_4 is a single-page HTML5 canvas game, **Sky Fighter**: a Raiden-style vertical shoot-em-up.

- Everything lives in `index.html`: inline CSS (`<style>`, ~lines 10–359), markup (~359–421), and one IIFE-wrapped `<script>` (~422–1667).
- `README.md` is a placeholder and has no setup instructions.
- There is no build step, package manifest, bundler, linter, or test suite. Do not invent npm/yarn commands.

## Running and verifying

- Open `index.html` directly in a browser, or serve the directory with any static server (for example `python3 -m http.server`).
- There are no automated tests. Verify changes by playing the game in a browser: start a run, fire, use a bomb, pause, clear a stage, fight a boss, and die.
- For headless checks, Playwright is pre-installed in the cloud environment. Launch Chromium with `chromium.launch()`, and use `executablePath: '/opt/pw-browsers/chromium'` only if a project pins its own Playwright version.

## Code architecture

All game logic is inside one closure in `index.html`. Key sections, in source order:

| Area | Notes |
| --- | --- |
| DOM refs | Cached `getElementById` lookups at the top of the IIFE. |
| Storage keys | `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion` (localStorage). |
| Adaptive quality | `trackPerf` samples frame time and steps `quality` 0–2 to reduce effects on slow devices. |
| Config tables | `DIFFICULTIES` (easy/normal/hard), `KILLS_PER_STAGE`, `THEMES` (per-stage sky palettes), `ENEMY_TYPES`, `BOSSES`. |
| Audio | Web Audio synthesis (`beep`, `noiseBurst`, `sfx.*`). No audio files. `ensureAudio()` must run from a user gesture. |
| Input | Keyboard (`keys` map keyed by `e.code`), pointer/touch joystick, and on-screen fire/bomb buttons. |
| Canvas sizing | `BASE_W=480`, `BASE_H=720` logical resolution, scaled to fit the window (capped at 1.6×). |
| Game state | `resetGame()`, `update(dt)`, `draw()`, and `loop(now)` drive a `requestAnimationFrame` loop. `dt` is clamped to 0.05s. |
| Weapons and bombs | `fireWeapon()`, `useBomb()`, `applyPowerUp()`. Weapon has 3 power levels; bombs clear the screen and damage bosses. |
| Bosses | `BOSS_PATTERNS` (radial, aimed spread, columns, spiral, cross, homing, double ring), `updateBoss`, `killBoss`. |
| Lifecycle | `loseLife()`, `endGame()`, `togglePause()`. |

### Controls

- Move: arrow keys or WASD, or the touch joystick.
- Fire: Space (held), or the fire button.
- Bomb: `X` or Shift, or the bomb button.
- Pause: `P` or Escape.
- Mute, reduced-motion, and pause are also available as on-screen buttons (`muteBtn`, `motionBtn`, `pauseBtn`).

## Conventions

- Keep the game as a single self-contained HTML file unless the user asks to split it. Do not add external CDN dependencies or a build step without asking.
- Keep the IIFE wrapper and `'use strict'`. Do not leak new globals.
- Use `const` and `let` with arrow functions, matching the existing style. Keep 2-space indentation.
- Tune gameplay through the config tables (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `THEMES`) rather than hard-coding magic numbers in update logic.
- Respect the `reducedMotion` setting for any new screen-shake, flash, or particle effect, and check `quality` before adding expensive per-frame work.
- Wrap all `localStorage` access in a way that tolerates it being unavailable (private browsing, blocked storage). The existing code does not yet do this, so new reads and writes should.
- Keep canvas drawing in `draw()` and state changes in `update()`. Do not mutate game state from draw helpers.

## Git workflow

- Develop on the branch assigned for the task (currently `claude/gifted-tesla-1aue3p`). Do not push to other branches without permission.
- Commit with clear, descriptive messages and push with `git push -u origin <branch>`.
- Do not open a pull request unless the user explicitly asks for one.
