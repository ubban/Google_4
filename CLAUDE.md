# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Google_4 is a small, dependency-free web game: **Sky Fighter**, a Raiden-style
vertical shoot-'em-up rendered on an HTML5 `<canvas>`.

The whole app lives in one file, `index.html`:

- `<style>` (lines ~10–359): all CSS, including the HUD, overlay card, touch
  controls, and the dark theme (`:root { color-scheme: dark; }`).
- Body markup (after `</style>`): HUD elements, overlay, touch controls,
  `#srLive` live region for screen readers.
- One `<script>` block (starts ~line 422) wrapping everything in an IIFE with
  `'use strict'`.

`README.md` is a placeholder and has no run instructions.

## Running and testing

- There is no build step, package manager, or test suite. Open `index.html`
  directly in a browser, or serve the directory with any static server.
- There are no automated tests. Verify gameplay changes by playing in a browser,
  checking both keyboard and touch input, and checking the console for errors.
- Do not add a build toolchain or external dependencies unless the user asks.
  The project is intentionally a single self-contained file.

## Code layout inside the script

Code is grouped with `// ---------- Section ----------` comments. Keep new code
in the matching section:

| Section | Contents |
| --- | --- |
| DOM refs | `const xEl = document.getElementById(...)` at the top |
| Storage keys | `STORAGE_*` constants (`skyfighter.*` namespace) |
| Adaptive performance | `trackPerf` lowers render quality (`quality` 0–2) when frame time is high |
| Tuning data | `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES`, `ENEMY_TYPES`, `BOSSES` |
| Audio | `ensureAudio`, `beep`, `noiseBurst`, and the `sfx` helpers (Web Audio, created lazily) |
| Input | `keys` map for keyboard, joystick for touch, mouse via canvas |
| Game state | `resetGame`, `player`, spawn helpers (`spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`) |
| Weapons & bombs | `fireWeapon`, `useBomb`, `applyPowerUp` |
| Bosses | `BOSS_PATTERNS` (bullet patterns), `updateBoss`, `killBoss` |
| Update / draw | `update(dt)`, `draw()`, `drawPlane`, `drawBossShip`, `drawClouds` |
| Loop | `loop(now)` drives `update` then `draw`; `dt` is clamped to 0.05 s |

## Game design facts

Keep these consistent when changing balance or content:

- **Canvas:** logical size is 480×720 (`BASE_W`, `BASE_H`). `resize()` scales
  the CSS size to fit the window, capped at 1.6×. Draw in logical coordinates.
- **Difficulty** (`easy` / `normal` / `hard`) sets spawn rate, enemy speed,
  enemy fire chance, starting lives, starting bombs, and boss HP multiplier.
- **Enemies:** `grunt` (stage 1+), `interceptor` (stage 1+, weaves), and `tank`
  (stage 2+, 4 HP). Type choice is weighted by the stage's available types.
- **Stages:** after `KILLS_PER_STAGE` (18) kills in a stage, a boss spawns.
  Defeating it increments `stage`, which picks the next theme from `THEMES` and
  the next boss from `BOSSES` (both indexed with `% length`).
- **Weapons:** `vulcan` and `laser`, each with power levels. Weapon-switch
  power-ups toggle between them.
- **Scoring:** the multiplier is `1 + min(9, floor(chain / 5))`, so it caps at
  10× and rises every 5 kills. The chain expires after 2.2 s without a kill
  (`breakChain`). Boss kills award a flat 500 base points.
- **Lives and bombs:** losing a life triggers invulnerability; `useBomb` clears
  the screen and damages bosses.
- **Controls:** mouse move/click, Arrows/WASD, Space fire, X or Shift bomb,
  P or Esc pause. Mobile uses a left-side drag joystick, FIRE, and BOMB buttons.

## Persistence

`localStorage` keys (all wrapped in the `skyfighter.` namespace):

- `skyfighter.bestScore`: high score
- `skyfighter.muted`: `'1'` when muted
- `skyfighter.difficulty`: difficulty name
- `skyfighter.reducedMotion`: `'1'` or `'0'`; defaults to the
  `prefers-reduced-motion` media query when unset

If you add a new setting, add a `STORAGE_*` constant next to the others and use
the same prefix. Keep the existing key names stable, since changing them
silently resets players' saved data.

## Conventions

- **Match the surrounding style.** The code uses 2-space indentation, single
  quotes, `const`/`let` (no `var`), arrow functions for callbacks, and short
  helper functions. Keep comment density similar to the nearby code.
- **Keep state in the IIFE.** Do not add globals. Do not attach handlers to
  `window` outside the existing input and resize sections.
- **Stay accessible.** Respect `reducedMotion` for shake, flash, and motion
  effects. Announce important state changes through `announce()` so screen
  readers hear them.
- **Stay mobile-friendly.** Preserve `touchControls` behavior and the viewport
  meta tag. Touch handlers use `{ passive: false }` where they call
  `preventDefault`.
- **Guard browser APIs.** `localStorage` and `AudioContext` can be unavailable
  or blocked. Keep the existing pattern of wrapping them, and make sure the
  game still runs without them.
- **Audio starts on user gesture.** Call `ensureAudio()` from input handlers.
  Browsers block audio that starts before a gesture.
- **Use `dt` for movement and timers.** Do not tie motion to frame count, so
  the game runs at the same speed on high-refresh displays.

## Git workflow

- Develop on the branch assigned for the session. Do not push to other
  branches without permission.
- Write commit messages in the imperative mood with a short summary line. Use a
  body for larger changes, as in the existing `Add Sky Fighter` commit.
- Do not open a pull request unless the user asks for one.
