# CLAUDE.md

Guidance for Claude Code when working in this repository.

## What this repo is

`Google_4` is a small static web project containing **Sky Fighter**, a Raiden-style vertical shoot-'em-up built as a single-page HTML5 canvas game.

| File | Purpose |
| --- | --- |
| `index.html` | The entire game: markup (~360 lines), CSS in a `<style>` block, and the game logic in one inline `<script>` IIFE. ~1,670 lines total. |
| `README.md` | Minimal project readme. |

There is no `package.json`, bundler, framework, test suite, linter, or CI config. Do not invent build or test commands. Don't add tooling unless asked.

## Running and verifying

- Open `index.html` directly in a browser, or serve the folder with any static server (e.g. `python3 -m http.server`).
- There are no automated tests. Verify changes by playing the game in a browser. For gameplay or layout changes, check at both desktop and phone-sized viewports, since the layout and touch controls differ.
- Keyboard, mouse, and touch input paths are all implemented. If you touch input code, test the paths you changed.

## Code structure (inside `index.html`)

- **DOM refs** (top of the IIFE): every element is looked up once by `id`. If you add a DOM element that the script uses, add its lookup there and keep the `id` in the markup.
- **Config tables** (near the top of the script):
  - `DIFFICULTIES`: spawn rates, enemy speed and fire chance, lives, bombs, and boss HP multiplier per difficulty (`easy` / `normal` / `hard`).
  - `KILLS_PER_STAGE`: kills needed to advance a stage.
  - `THEMES`: background sky palettes, one per stage, cycled.
  - `ENEMY_TYPES`: `grunt`, `interceptor`, `tank`, with size, HP, speed, colour, score, and `minStage`.
  - `BOSSES`: boss names, colours, and HP multiplier. `BOSS_PATTERNS` holds the attack pattern functions.
- **Systems**:
  - Audio: a Web Audio synth (`ensureAudio`, `beep`, `noiseBurst`, the `sfx` object). Audio starts only after a user gesture. Respect `muted`.
  - Adaptive quality: `trackPerf` lowers the `quality` level when frame time is high.
  - Input: `keys` for keyboard, `joystick*` for touch, `canvasPointFromEvent` for mouse.
  - Game loop: `update(dt)` mutates state, `draw()` renders, and `loop(now)` drives both through `requestAnimationFrame`.
  - Game state: `resetGame`, `loseLife`, `endGame`, `togglePause`, `useBomb`, `fireWeapon`, `applyPowerUp`.
- **Persistence**: `localStorage` keys are prefixed `skyfighter.` (`bestScore`, `muted`, `difficulty`, `reducedMotion`). Reads and writes that touch storage are wrapped in `try`, so the game still works when storage is blocked. Keep that pattern for any new key.
- **Accessibility**: the `announce()` helper writes to the `#srLive` live region. Use it for important game events. The reduced-motion toggle adds the `reduced-motion` body class and gates screen shake, flashes, and pulses. Honour that toggle for any new visual effect.

## Conventions

- Keep everything in `index.html` unless there is a clear reason to split it. The single-file design is intentional.
- Match the existing style: `const` / `let`, arrow functions for callbacks, 2-space indentation, single quotes in JS, and `'use strict'` inside the IIFE.
- Don't introduce external network dependencies (CDNs, fonts, analytics). The game should run offline from one file.
- Keep the canvas logical size at `BASE_W` × `BASE_H` (480 × 720). `resize()` scales it to the viewport. Draw in logical coordinates.
- Game-balance numbers belong in the config tables, not scattered through the logic.
- Don't leave debug output (`console.log`) or commented-out code in committed changes.

## Git workflow

- Default working branch for this session: `claude/gifted-tesla-gbcbp8`. Commit and push there.
- Do not open a pull request unless explicitly asked.
- Commit messages: a short imperative summary line, then a body explaining the why when it isn't obvious.
