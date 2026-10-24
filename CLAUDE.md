# CLAUDE.md

Guidance for Claude Code when working in this repository.

## What this repo is

A small static project containing **Sky Fighter**, a Raiden-style vertical shoot-em-up
built as a single-page HTML5 canvas game.

- `index.html` — the entire game: markup, CSS (`<style>`, ~lines 10–359), and one
  inline `<script>` IIFE (~lines 422–1669) holding all logic.
- `README.md` — placeholder; no real project docs yet.

There is no build step, no package manager, no test suite, and no linter configured.
Do not add tooling unless the user asks for it.

## Running it

Open `index.html` in a browser, or serve the directory with any static server
(e.g. `python3 -m http.server`). No dependencies are fetched.

## Code layout inside the script

The game code lives in a single `(function () { 'use strict'; ... })();` block. Keep
new code inside it so nothing leaks onto `window`. Sections are marked with
`// ---------- Name ----------` comments; follow that convention when adding one.

Key sections, in file order:

- **DOM refs and storage keys** — `getElementById` lookups at the top; `localStorage`
  keys are prefixed `skyfighter.` (`bestScore`, `muted`, `difficulty`, `reducedMotion`).
- **Adaptive performance** — `trackPerf` lowers/raises a `quality` level (0–2) from
  rolling frame times.
- **Config tables** — `DIFFICULTIES`, `THEMES` (stage backgrounds), `ENEMY_TYPES`,
  `BOSSES`, `BOSS_PATTERNS`, `KILLS_PER_STAGE`. Balance changes usually belong here
  rather than in logic.
- **Audio** — `ensureAudio`, `beep`, `noiseBurst`, and the `sfx` object. Audio is
  created lazily on first user gesture because browsers require one.
- **HUD helpers** — hearts, bombs, combo popups, stage banner, screen shake, flashes.
- **Input** — keyboard (`keys[e.code]`), touch joystick and buttons, mouse on canvas.
- **Game state** — `resetGame`, `update(dt)`, spawning, collisions, weapons, power-ups.
- **Bosses** — `updateBoss`, `killBoss`, and the fire-pattern functions.
- **Rendering** — `draw()` and the plane/boss/cloud drawing helpers.
- **Loop** — `loop(now)` clamps `dt` to 50 ms and runs `update` then `draw` only while
  running and not paused.

## Gameplay facts worth knowing

- Controls: arrows or WASD to move; Space or hold the fire button to shoot;
  X / Shift (or the bomb button) for a bomb; P or Escape to pause.
- Difficulty (`easy` / `normal` / `hard`) sets spawn rate, enemy shot chance, lives,
  bombs, and boss HP multiplier.
- Each stage ends after `KILLS_PER_STAGE` kills, which spawns a boss. Defeating the
  boss advances `stage`, and the background cycles through `THEMES` via
  `(stage - 1) % THEMES.length`.
- Score uses a kill-chain multiplier (`addScore`, `breakChain`).

## Conventions

- Match the existing style: 2-space indent, single quotes in JS, `const` by default,
  arrow functions for callbacks, short comments only where the intent is not obvious.
- Keep everything in `index.html` unless there is a clear reason to split files. If a
  split happens, update this file.
- Preserve the accessibility and mobile work already in place: the `aria-live` region
  (`announce`), the reduced-motion toggle and `prefers-reduced-motion` check, and the
  touch controls. Test any input change on both keyboard and touch paths.
- Preserve the `localStorage` keys above so saved best scores and settings survive.
  Wrap new storage reads in a fallback, since storage can be unavailable.

## Verifying changes

There are no automated tests. For any gameplay or rendering change:

1. Open `index.html` in a browser (or a headless Chromium) and check the console for errors.
2. Play through start, pause/resume, a bomb, losing a life, and game over.
3. For balance or boss changes, check at least `normal` and `hard`.
4. For UI changes, check both a wide desktop viewport and a phone-sized viewport.

If you cannot run the game in the environment, say so explicitly in the summary rather
than implying it was tested.

## Git

- Development branch for this work: `claude/gifted-tesla-7yqjav`.
- Keep commits focused and write messages that explain the gameplay or code reason for
  the change.
- Do not open a pull request unless the user asks for one.
