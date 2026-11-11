# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a single-page browser game, **Sky Fighter**: a Raiden-style vertical shoot-'em-up.

- The entire game lives in `index.html` (~1,670 lines): inline `<style>`, the HTML markup, and one inline `<script>` IIFE.
- `README.md` is a placeholder and contains no setup instructions.
- There is no build step, package manager, bundler, framework, or test suite. Nothing needs installing.

## Running the game

Open `index.html` directly in a browser, or serve the directory with any static server, for example:

```bash
python3 -m http.server 8000   # then visit http://localhost:8000
```

There are no automated tests or linters configured. Verify changes by playing the game in a browser, checking the browser console for errors, and exercising the affected feature.

## Code layout (`index.html`)

- **Markup**: `#container` holds the `<canvas id="gameCanvas">`, the HUD (`#hud`: score, best, multiplier, weapon, hearts, bombs), the boss bar, the top-right icon buttons (`#muteBtn`, `#motionBtn`, `#pauseBtn`), touch controls (`#touchControls`), and the start/overlay card (`#overlay`).
- **Script**: a single `(function () { 'use strict'; ... })();` IIFE. Nothing is exported to `window`; keep new code inside the IIFE.
- **Element lookup**: DOM references are captured once at the top of the script via `getElementById`. Add new references there rather than querying inside the loop.
- **Config tables** (top of script, around lines 463–530):
  - `DIFFICULTIES`: `easy` / `normal` / `hard`, controlling spawn rate, enemy speed, enemy fire chance, starting lives, starting bombs, and boss HP multiplier.
  - `KILLS_PER_STAGE` (18): kills needed to advance a stage.
  - `THEMES`: background sky gradients, cloud and sun colors, and theme names. Cycled per stage.
  - `ENEMY_TYPES`: `grunt`, `interceptor` (weaving), and `tank` (4 HP, unlocks at stage 2), with score values.
  - `BOSSES`: six bosses (VANGUARD → TITAN), each with colors and an HP multiplier.
  - `BOSS_PATTERNS` (around line 1372): attack patterns, each built from helpers such as `fireRadial`, `fireAimedSpread`, `fireSpiralBurst`, `fireCross`, `fireHomingSeeded`, and `fireDoubleRing`.
- **Core loop**: `update(dt)` advances game state, `draw()` renders the frame, and `loop(now)` calls both via `requestAnimationFrame`. `dt` is clamped to 50 ms so a backgrounded tab cannot cause a large jump.
- **Adaptive quality**: `trackPerf` averages the last 45 frame times and steps `quality` between 0 (full), 1 (medium), and 2 (low) to keep frame rate up. Rendering code should respect `quality`.
- **Audio**: a Web Audio synth (`ensureAudio`, `beep`, `noiseBurst`, and the `sfx` object). Audio is created lazily on first user gesture (`ensureAudio()`), which browsers require. Mute is persisted.
- **Accessibility**: `announce()` writes to the `#srLive` aria-live region. The "reduced motion" toggle turns off screen shake and flash/pulse effects, and defaults to the OS `prefers-reduced-motion` setting until the user chooses.
- **Scaling**: the canvas uses a logical size of `BASE_W = 480` × `BASE_H = 720` and is resized to fit the window in `resize()`.

## Persistence

`localStorage` keys (all prefixed `skyfighter.`):

| Key | Value |
| --- | --- |
| `skyfighter.bestScore` | High score (number) |
| `skyfighter.muted` | `'1'` when muted |
| `skyfighter.difficulty` | `easy`, `normal`, or `hard` |
| `skyfighter.reducedMotion` | `'1'` or `'0'`; absent means use OS preference |

Do not rename these keys without a migration. Existing players would lose saved data.

## Controls

- **Desktop**: mouse to steer, click to fire; arrow keys or WASD to move; Space to fire; X or Shift for bomb; P or Esc to pause.
- **Mobile**: drag the left zone to fly, tap FIRE to shoot, tap BOMB to use a smart bomb.

Keyboard handling is in `keydown`/`keyup` listeners that populate the `keys` map, keyed by `e.code`.

## Conventions

- Keep everything in `index.html` unless there is a clear reason to split files. If the project grows, update this section.
- Match the existing style: 2-space indent, `const`/`let`, single quotes, arrow functions for callbacks, and named `function` declarations for game helpers.
- Put tunable numbers in the config tables (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, etc.) rather than hard-coding them in logic.
- Use `ctx` drawing calls inside `draw()` or its helpers; do not touch the canvas from `update()`.
- Respect `reducedMotion` for any new screen-shake, flash, or pulse effect, and respect `quality` for expensive effects.
- Any new user-visible state change should be announced through `announce()` when it matters to screen-reader users.

## Git

- Development branch for the current task: `claude/gifted-tesla-3ij50d`.
- Commit messages should be short and imperative, for example `Add Sky Fighter: ...`.
- Do not open pull requests unless explicitly asked.
