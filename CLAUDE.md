# CLAUDE.md

Guidance for Claude Code when working in this repository.

## What this repo is

`Google_4` is a small static project containing a single browser game, **Sky Fighter** — a Raiden-style vertical shoot-em-up.

- `index.html` — the entire game: inline CSS (`<style>`), markup (`<canvas id="gameCanvas">` plus overlay/HUD/touch controls), and one inline `<script>` (~1,250 lines) wrapped in an IIFE with `'use strict'`.
- `README.md` — a placeholder with only the title. Not a source of truth for behavior.

There is no `package.json`, no bundler, no framework, no test suite, no linter, and no CI configuration. Nothing needs to be installed.

## Running and verifying

- Run it by opening `index.html` directly in a browser (or serve the folder with any static file server, e.g. `python3 -m http.server`).
- There are no automated tests or build steps. Verify changes by playing the game in a browser. For headless checks, Playwright/Chromium is available in the cloud environment; load `file://…/index.html` and drive it there.
- Do not add a build system, package manager, or framework unless asked. The project is intentionally a single self-contained file.

## Code structure inside `index.html`

The script is organized as commented sections. Keep new code in the matching section:

| Section (comment) | Contents |
|---|---|
| Adaptive performance | `trackPerf` — drops render `quality` (0 full / 1 medium / 2 low) when frame time is high |
| Constants | `DIFFICULTIES`, `KILLS_PER_STAGE` (18), `THEMES` (6 sky palettes, cycled per stage), `ENEMY_TYPES` (`grunt`, `interceptor`, `tank`), `BOSSES` (6 bosses, cycled), `BOSS_PATTERNS` |
| Audio | Web Audio synth: `ensureAudio`, `beep`, `noiseBurst`, `sfx.*`. Muted state is persisted |
| Input | Keyboard (`keys[e.code]`), mouse on the canvas, and touch joystick + FIRE/BOMB buttons |
| Game state | `resetGame`, `update(dt)`, `draw()`, spawning, power-ups, bomb, boss logic, `loseLife`, `endGame`, `togglePause` |
| Main loop | `loop(now)` — `dt` is capped at 0.05 s; `update` and `draw` only run when `running && !paused` |

Other notes:

- **Fixed logical resolution:** the canvas is always `480×720` (`BASE_W`/`BASE_H`). `resize()` only scales the CSS size. Game coordinates should stay in these units.
- **Persistence:** `localStorage` keys are `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, and `skyfighter.reducedMotion`. Reads are not wrapped in try/catch today; if you add or change storage, guard it, since storage can throw in private windows or when site data is blocked.
- **Accessibility hooks:** `announce()` writes to the `#srLive` aria-live region; `reduced-motion` is applied as a body class and honors `prefers-reduced-motion` unless the user has set a preference. Keep these working when adding effects: `screenShake()` already scales magnitude and duration down when `reducedMotion` is on, so new shake/flash effects should follow the same pattern.
- **Difficulty** (`easy`/`normal`/`hard`) changes spawn rate, enemy speed, enemy fire chance, starting lives, starting bombs, and boss HP multiplier.
- **Controls** (also shown in-game in `#controlsHint`): mouse moves/fires, Arrow keys or WASD move, Space fires, X or Shift uses a bomb, P or Esc pauses. Touch: drag left to fly, tap FIRE, tap BOMB.

## Conventions

- Plain HTML/CSS/JavaScript, ES2017+ features are fine. No transpilation, no modules, no external dependencies (no CDN scripts or fonts are currently loaded — keep it that way unless asked).
- Keep everything in `index.html`. Do not split files without a reason.
- Match the existing style: 2-space indentation, single quotes in JS, `const`/`let`, camelCase identifiers, UPPER_SNAKE for tuning constants.
- Put tuning values (speeds, spawn rates, HP, colors) in the constants tables rather than hard-coding them in logic.
- Keep the game's `dt`-based movement: multiply per-frame motion by `dt` so behavior is frame-rate independent.

## Git and workflow

- Default working branch for this session: `claude/gifted-tesla-ekuh5k`. Commit and push there (`git push -u origin <branch>`); do not push to other branches without explicit permission.
- Do not open a pull request unless explicitly asked.
- Keep commits focused. Commit messages describe the gameplay or code change, not the process.
