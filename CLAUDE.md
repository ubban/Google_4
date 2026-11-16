# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a small static web project. Its main deliverable is **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs in a browser canvas.

- `index.html` — the whole game: markup, CSS, and JavaScript in one file (~1,670 lines).
- `README.md` — minimal placeholder; no setup instructions yet.

There is no build step, no package manager, no bundler, no framework, and no test suite. Do not introduce any of these without asking.

## Running the game

- Open `index.html` directly in a browser, or serve the directory with any static server (for example `python3 -m http.server` and browse to `index.html`).
- The game needs no network access and no external assets.

## Architecture of `index.html`

Layout, top to bottom:

1. `<style>` (~lines 10–360): all CSS, including the HUD, overlay, touch controls, and `body.reduced-motion` rules.
2. `<body>` markup (~lines 363–420): the `<canvas id="gameCanvas">`, HUD elements, the start/pause overlay (`#overlay`), touch controls, and a screen-reader live region (`#srLive`).
3. One `<script>` block (~line 422 onward) wrapped in an IIFE with `'use strict'`. Nothing is exposed on `window`. Keep it that way.

Inside the script, the sections are roughly in this order:

- **DOM refs and constants** — element lookups, `STORAGE_*` keys, `BASE_W = 480`, `BASE_H = 720`.
- **Adaptive performance** (`trackPerf`) — samples frame time and steps a `quality` level (0–2) up or down. Quality is used to reduce effects on slow devices.
- **Tuning tables** — `DIFFICULTIES` (spawn rates, enemy speed, shot chance, lives, bombs, boss HP multiplier), `THEMES` (background palettes, cycled per stage), `ENEMY_TYPES` (`grunt`, `interceptor`, `tank`), `BOSSES` (six boss definitions, cycled), `KILLS_PER_STAGE = 18`.
- **Audio** — `ensureAudio`, `beep`, `noiseBurst`, and the `EXPLODE_SFX` mapping. Audio is created lazily on the first user gesture (`ensureAudio` is called from input handlers) to satisfy browser autoplay rules.
- **HUD and feedback** — `renderHearts`, `renderBombs`, `flashHit`, `flashBomb`, `showComboPop`, `showStageBanner`, `screenShake`.
- **Layout and input** — `resize()` scales the canvas to fit the viewport while keeping the 480×720 logical resolution. Keyboard (`keys` map), mouse/pointer steering and firing, and a virtual joystick and buttons for touch.
- **Game state** — `resetGame`, `update(dt)`, `fireWeapon`, `useBomb`, `applyPowerUp`, `addScore` (with a combo multiplier, reset by `breakChain`), `loseLife`, `endGame`, `togglePause`.
- **Bosses** — `spawnBoss`, `BOSS_PATTERNS` (radial, spread, columns, spiral, cross, homing, double-ring bullet patterns), `updateBoss`, `killBoss`.
- **Rendering** — `draw()`, `drawPlane`, `drawBossShip`, `drawClouds`. Drawing is plain Canvas 2D; there is no WebGL.
- **Loop** — `loop(now)` clamps `dt` to 50 ms, then calls `update` and `draw` only when running and not paused. Started with `requestAnimationFrame`.

## Persistence

The game stores these keys in `localStorage`. Changing a key name will silently reset that setting for existing players, so treat the names as a public interface:

| Key | Meaning |
| --- | --- |
| `skyfighter.bestScore` | Best score (number) |
| `skyfighter.muted` | `'1'` when sound is muted |
| `skyfighter.difficulty` | `easy`, `normal`, or `hard` |
| `skyfighter.reducedMotion` | `'1'` / `'0'`; if absent, the `prefers-reduced-motion` media query decides |

Reads of `localStorage` are not wrapped in `try/catch` today. If you add new storage, guard it, because some browsers throw in private mode or when storage is blocked.

## Controls (keep in sync with the in-game hint)

- Move: mouse/pointer, Arrow keys, or WASD
- Fire: click/hold, or Space
- Bomb: `X` or Shift
- Pause: `P` or Esc
- Touch: drag on the left to fly, FIRE and BOMB buttons on the right

If you change a control, update the `#controlsHint` paragraph (~line 417) too.

## Conventions

- **Single file.** Keep the game in `index.html` unless the user asks for a split. Do not add external CDN scripts or fonts.
- **Match the existing style:** 2-space indent, single quotes in JS, `const`/`let` (no `var`), arrow functions for callbacks, and section headers in the form `// ---------- Name ----------`.
- **Keep the IIFE and `'use strict'`.** New helpers should be declared inside the IIFE.
- **Tune numbers in the tables** (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `THEMES`) rather than hard-coding magic numbers in update/draw code.
- **Respect the logical resolution.** Gameplay coordinates are in the 480×720 space; do not use CSS pixels for game logic.
- **Accessibility:** announce important state changes through `announce()` (which feeds `#srLive`), and honor `reducedMotion` for screen shake and flashes.

## Verification

No automated tests or linters exist. When you change gameplay or rendering:

1. Open the page in a browser (or serve it statically) and check the console for errors.
2. Play through at least one stage on each difficulty you touched, and exercise pause, bomb, and game over.
3. For layout or input changes, check both a desktop viewport and a narrow mobile viewport, including the touch controls.

If you add a test suite or tooling, document the exact commands here.

## Git

- Default branch work is done on the branch the session specifies; do not push to other branches without permission.
- Keep commits focused and describe the gameplay or file affected in the message.
