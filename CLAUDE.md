# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo contains **Sky Fighter**, a Raiden-style vertical shoot-em-up built as a
single-file HTML5 canvas game.

- `index.html` — the entire game: markup (`<body>`), CSS (`<style>`, ~lines 10–360),
  and one inline IIFE in `<script>` (~line 422 onward). There is no build step.
- `README.md` — minimal project readme.

There are no npm dependencies, no bundler, no framework, and no test suite.

## Running and verifying

- Open `index.html` directly in a browser, or serve the directory with any static
  server (e.g. `python3 -m http.server`) and visit it.
- There are no automated tests or linters configured. Verify changes by loading the
  page and playing; for UI or gameplay changes, check desktop and mobile layouts
  (touch controls appear on touch devices).
- Use the Chromium/Playwright setup in the environment if a headless check is needed.

## Code structure (inside `index.html`)

The script is a single `(function () { 'use strict'; ... })()` block. Major sections,
in source order:

1. **DOM refs and constants** — element lookups, `STORAGE_*` localStorage keys.
2. **Adaptive performance** — `trackPerf` steps a `quality` level (0–2) from frame times.
3. **Tuning tables** — `DIFFICULTIES` (easy/normal/hard), `KILLS_PER_STAGE`, `THEMES`
   (background palettes cycled per stage), `ENEMY_TYPES`, `BOSSES`.
4. **Audio** — `ensureAudio`, `beep`, `noiseBurst`, and the `sfx` table. Audio is
   created lazily on the first user gesture (browser autoplay policy).
5. **HUD helpers** — hearts/bombs rendering, flashes, combo pops, stage banner, shake.
6. **Canvas sizing** — `BASE_W = 480`, `BASE_H = 720`, scaled in `resize()`.
7. **Input** — keyboard (`keys` map), mouse/pointer steering, touch joystick and
   FIRE/BOMB buttons.
8. **Game state** — `resetGame()`, `update(dt)`, spawning, collisions, weapons
   (`fireWeapon`, Vulcan/Laser with 3 power levels), bombs (`useBomb`).
9. **Bosses** — `BOSS_PATTERNS` (radial, spread, spiral, cross, homing, etc.),
   `updateBoss`, `killBoss`.
10. **Lifecycle** — `loseLife`, `endGame`, `togglePause`.
11. **Rendering** — `drawPlane`, `drawBossShip`, `drawClouds`, `draw()`.
12. **Main loop** — `loop(now)` driven by `requestAnimationFrame`.

Keep new code in the matching section and reuse the existing tuning tables rather than
hard-coding numbers in the update or draw paths.

## Controls (for reference when testing)

- Keyboard: Arrows / WASD move, Space fire, X or Shift bomb, P or Esc pause.
- Mouse: move to steer, click to fire.
- Touch: drag left zone to fly, FIRE and BOMB buttons on screen.

## Persistence

Stored in `localStorage` under the `skyfighter.*` prefix:

- `skyfighter.bestScore`
- `skyfighter.muted`
- `skyfighter.difficulty`
- `skyfighter.reducedMotion`

Reduced motion defaults to the `prefers-reduced-motion` media query until the user
sets it. Renaming these keys will reset player data, so treat them as a stable contract.

## Conventions

- Keep the game as a single self-contained HTML file unless the user asks to split it.
  Do not introduce a build step or external dependencies without discussion.
- Keep the code in the existing style: `const`/`let`, 2-space indentation, single
  quotes, no semicolon-less code, and comments as `// ---------- Section ----------`
  dividers.
- Accessibility matters here: preserve the `aria-live` announcements (`announce()`),
  the reduced-motion path, and keyboard reachability of the buttons.
- Preserve mobile behavior (touch joystick, `touch-action` handling, safe-area
  layout) when changing the HUD or overlays.
- Do not change the `localStorage` key names or the difficulty table semantics without
  calling it out explicitly.

## Git

- Develop on the branch assigned for the session. Do not push to other branches
  without explicit permission.
- Commit messages: a short imperative summary line, then a body explaining the
  gameplay or code change where useful.
