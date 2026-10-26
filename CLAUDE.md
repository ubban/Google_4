# CLAUDE.md

Guidance for working in this repository.

## What this is

"Sky Fighter" — a Raiden-style vertical shoot-'em-up that runs in the browser as a
single self-contained file, `index.html`. All HTML, CSS, and JavaScript live in that
file; there are no external dependencies, no bundler, and no package manager.

`README.md` is a placeholder and carries no project documentation.

## Running and testing

- Open `index.html` directly in a browser, or serve the directory with any static
  server. No build step is needed.
- There is no test suite, linter, or CI configuration in the repo. Verify changes by
  playing the game in a browser (desktop and, if touch code is touched, a touch device
  or touch emulation).

## File layout (`index.html`)

- `<style>` (~lines 10–359): all styling, including the HUD, overlay, touch controls,
  and the `.reduced-motion` mode.
- `<body>` (~lines 362–420): the DOM the script hooks into. Key ids: `gameCanvas`,
  `hud`, `overlay`, `startBtn`, `diffRow` (`.diffBtn` buttons), `touchControls`,
  `bossBarWrap`, `stageBanner`, `comboPop`, `srLive` (screen-reader live region).
- `<script>` (~lines 422–1667): one IIFE wrapping the whole game, with `'use strict'`.
  Nothing is exposed on `window`; keep it that way unless there is a reason.

Rough order of sections inside the IIFE:

1. DOM lookups and `STORAGE_*` keys
2. Adaptive quality (`trackPerf`, `quality` 0–2)
3. Tuning tables: `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES`, `ENEMY_TYPES`, `BOSSES`
4. Settings load and UI helpers (`updateMotionBtn`, `updateMuteBtn`, `setDifficulty`)
5. Audio (`ensureAudio`, `beep`, `noiseBurst`) — WebAudio synthesis, no audio files
6. Layout and input (`resize`, joystick, mouse/touch point mapping)
7. Game state (`resetGame`), spawning, power-ups, bombs, weapons (`fireWeapon`)
8. `update(dt)` — per-frame simulation; `draw()` — per-frame rendering
9. Bosses (`BOSS_PATTERNS`, `updateBoss`, `killBoss`), life loss, `endGame`, pause
10. `loop(now)` — `requestAnimationFrame` driver, with `dt` clamped to 0.05 s

## Conventions

- **Keep it a single file.** New features go into `index.html` unless the user asks
  for a split. Don't introduce a build step or external CDN dependencies without
  asking.
- **Match the existing style:** 2-space indent, single quotes in JS, `const`/`let`,
  arrow functions for callbacks, short section comments with `// ---------- Name ----------`.
- **Time is in seconds, scaled by `dt`.** Movement and timers multiply by `dt`.
  Spawn timing uses a frame-based `spawnTimer` decremented by `dt * 60`. Don't mix the two.
- **Tuning lives in the tables.** Add or adjust enemies, bosses, themes, and difficulty
  values in the data objects rather than hard-coding numbers in logic.
- **Canvas is a fixed logical size.** `BASE_W = 480`, `BASE_H = 720`; `resize()` scales
  it to the window. Coordinates in game logic use the canvas pixel space.

## Persistence

`localStorage` keys (all prefixed `skyfighter.`):

| Key | Meaning |
| --- | --- |
| `skyfighter.bestScore` | Best score (number) |
| `skyfighter.muted` | `'1'` when audio is muted |
| `skyfighter.difficulty` | `easy` / `normal` / `hard` |
| `skyfighter.reducedMotion` | `'1'` / `'0'`; if absent, follows `prefers-reduced-motion` |

Every `localStorage` read/write should tolerate failure (private mode, blocked
storage). If you add a key, prefix it with `skyfighter.` and document it here.

## Accessibility and input

- Reduced motion: when enabled, flash effects are skipped and screen shake is dampened
  (magnitude × 0.15, capped at 0.1 s). Any new visual effect that shakes or flashes must
  respect `reducedMotion`.
- Announcements for assistive tech go through `announce(text)` into `#srLive`.
- Controls: mouse (steer and click to fire), keyboard (arrows/WASD, Space fire, X or
  Shift bomb, P or Esc pause), and touch (left-side joystick, FIRE and BOMB buttons).
  Keep all three working when changing input code.

## Working on this repo

- Develop on the branch named in the session instructions; do not push to other branches
  without permission.
- Commit with clear messages describing the gameplay or code change.
- Because there are no automated checks, state in commit or PR text what was verified
  by hand and in which browser.
