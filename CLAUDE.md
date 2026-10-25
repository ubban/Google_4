# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Google_4 is a small static web project. Its only game is **Sky Fighter**, a
Raiden-style vertical shoot-'em-up rendered on an HTML5 `<canvas>`.

- `index.html` — the entire game: markup, CSS (`<style>`), and JavaScript
  (`<script>`) in one file (~1,670 lines).
- `README.md` — a one-line title. Keep it short.

There is no build step, package manager, test suite, or linter. Do not add
one unless asked.

## Running the game

Open `index.html` directly in a browser, or serve the directory with any
static server (for example `python3 -m http.server`). No install is needed.

## Code layout (inside `index.html`)

The game script is a single IIFE (`(function () { 'use strict'; ... })();`).
Keep new code inside it so nothing leaks onto `window`.

Roughly in order of appearance:

1. **DOM references** — `const xxxEl = document.getElementById(...)` at the top.
2. **Storage keys** — `skyfighter.*` prefixed `localStorage` keys (best score,
   mute, difficulty, reduced motion).
3. **Adaptive performance** — `trackPerf` lowers/raises a `quality` level
   (0 full, 1 medium, 2 low) based on average frame time.
4. **Tuning tables** — `DIFFICULTIES`, `THEMES` (per-stage sky palettes),
   `ENEMY_TYPES`, `BOSSES`, and `KILLS_PER_STAGE`. Balance changes belong here
   rather than in the logic.
5. **Audio** — Web Audio helpers (`ensureAudio`, `beep`, `noiseBurst`). Audio is
   created lazily on the first user gesture.
6. **HUD and effects** — hearts/bombs rendering, combo and stage banners,
   screen shake, hit flash.
7. **Layout and input** — `resize` (fixed `BASE_W` × `BASE_H` = 480 × 720,
   scaled to fit), keyboard handlers, and virtual joystick / fire / bomb
   buttons for touch devices.
8. **Game state** — `resetGame`, `update(dt)`, spawning, collisions
   (`rectsOverlap`), scoring with a kill-chain multiplier, weapons
   (`fireWeapon`), bombs (`useBomb`), and power-ups (`applyPowerUp`).
9. **Bosses** — `BOSS_PATTERNS` are bullet-pattern functions, selected by
   `updateBoss`. `killBoss` advances the stage.
10. **Rendering** — `draw()` and the `drawPlane` / `drawBossShip` /
    `drawClouds` helpers. Drawing is pure canvas 2D; there are no sprite files.
11. **Main loop** — `loop(now)` clamps `dt` to 50 ms, updates and draws only
    while running and not paused, then calls `requestAnimationFrame`.

## Conventions

- **Vanilla JS, ES2017+ style.** No frameworks, no bundler, no external
  dependencies. Use `const` by default and arrow functions for callbacks, as
  the existing code does.
- **Match the surrounding style.** The code uses 2-space indentation, single
  quotes in JS, short section banners such as `// ---------- Input ----------`,
  and aligned object literals in the tuning tables. Follow those patterns.
- **Keep it in one file** unless the user asks for a split. The single-file
  layout is deliberate for easy hosting.
- **Persist only through the existing `skyfighter.*` keys.** Wrap new
  storage reads and writes so the game still works when `localStorage` is
  unavailable (private mode, blocked storage).
- **Respect reduced motion.** Effects that shake or flash should check
  `reducedMotion` and the `reduced-motion` body class.
- **Accessibility.** Announce important state changes through the `srLive`
  live region via `announce(text)`.
- **Mobile.** Any new control needs a touch path as well as keyboard input.
  Use `{ passive: false }` plus `preventDefault()` on touch handlers that
  must suppress scrolling.

## Gameplay reference

| Input (keyboard) | Action |
| --- | --- |
| Arrow keys / WASD | Move |
| Space (hold) | Fire |
| X or Shift | Bomb (clears the screen and damages bosses) |
| P or Esc | Pause / resume |

Difficulty (`easy`, `normal`, `hard`) changes lives, bombs, spawn rate, enemy
speed, enemy fire chance, and boss HP multiplier.

Weapons: **Vulcan** and **Laser**. A weapon power-up raises the weapon level
(1–3); once at level 3, the next one switches to the other weapon and resets
to level 1. Other power-ups add a bomb or an extra life.

## Making changes

- Make the smallest change that does the job. Do not reformat unrelated code.
- After editing `index.html`, verify by loading it in a browser and playing
  through at least one stage. Check the console for errors. A passing
  syntax check alone is not enough for gameplay changes.
- When changing balance numbers, update the relevant tuning table and mention
  the old and new values in the commit message.
- Work on the designated feature branch. Do not push to `main` directly.
