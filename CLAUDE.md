# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo contains **Sky Fighter**, a Raiden-style vertical shoot-em-up built as a
single-page HTML5 canvas game. The whole game lives in `index.html`: markup, CSS,
and one inline `<script>` IIFE. There is no build step, no bundler, no package
manager, and no test suite.

Files:

- `index.html` — the entire game (about 1,670 lines).
- `README.md` — a stub. Keep it short; put developer documentation here instead.
- `CLAUDE.md` — this file.

## Running the game

Open `index.html` directly in a browser, or serve the directory with any static
server (for example `python3 -m http.server`). No install step is needed.

There are no automated tests, linters, or build commands configured. Verify
changes by playing the game in a browser, covering at least:

- Start, play, lose all lives, and reach the game-over overlay.
- Pause/resume (`P`/`Esc` or the pause button).
- A stage transition and a boss fight.
- Touch controls, if the change touches input or layout.

## Code layout inside `index.html`

The file is organized in this order. Keep new code in the matching section.

1. **`<style>`** — CSS for the HUD (pill-style stats, hearts, bombs), the glass
   overlay card, touch controls, boss health bar, and the `.reduced-motion` class.
2. **`<body>`** — the DOM: `#container` holds the canvas, the HUD, the boss bar,
   pop-up/banner elements, the top-bar buttons, touch controls, and `#overlay`
   (title, difficulty buttons, start button, control hints).
3. **`<script>`** — one IIFE with `'use strict'`. Inside it:
   - DOM lookups at the top (`const xEl = document.getElementById(...)`).
   - Persistence keys: `STORAGE_*` constants prefixed `skyfighter.`.
   - Tuning tables: `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES`, `ENEMY_TYPES`,
     `BOSSES`, and `BOSS_PATTERNS`.
   - Audio: `ensureAudio`, `beep`, `noiseBurst`, and the `sfx` object.
   - HUD helpers: `renderHearts`, `renderBombs`, `showComboPop`, `showStageBanner`,
     `screenShake`, `flashHit`, `flashBomb`.
   - Input: keyboard (`keys` map), mouse/pointer, and touch joystick plus
     FIRE/BOMB buttons.
   - Sizing: `BASE_W = 480`, `BASE_H = 720`, scaled to fit the viewport in `resize()`.
   - Game logic: `resetGame`, `update(dt)`, `spawnEnemy`, `spawnBoss`,
     `maybeSpawnPowerUp`, `fireWeapon`, `useBomb`, `applyPowerUp`, `loseLife`,
     `endGame`, `togglePause`.
   - Boss patterns: `fireRadial`, `fireAimedSpread`, `fireColumns`,
     `fireSpiralBurst`, `fireCross`, `fireHomingSeeded`, `fireDoubleRing`, and
     the `BOSS_PATTERNS` array consumed by `updateBoss`.
   - Rendering: `drawPlane`, `drawBossShip`, `drawClouds`, `draw()`.
   - Main loop: `loop(now)` caps `dt` at 0.05 s, runs `trackPerf`, then
     `update` and `draw`. Start is wired on `startBtn` near the end of the
     IIFE, and pause on `pauseBtn` just above the loop.

## Game design conventions

- **Fixed logical resolution.** All game coordinates use the 480×720 base. The
  canvas is scaled for display only; do not hard-code screen pixels in game logic.
- **Time-based updates.** Movement and timers use `dt` in seconds. Keep new
  behavior frame-rate independent.
- **Data-driven tuning.** Balance numbers belong in the tables near the top of
  the script (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `THEMES`), not inlined
  in update logic. Add a new enemy or boss by adding a table entry first.
- **Difficulty is a preset.** Each difficulty sets spawn rates, enemy speed,
  enemy fire chance, starting lives, bombs, and a boss HP multiplier. Read
  these through `DIFFICULTIES[difficulty]`.
- **Power-ups.** Pickups are `{ x, y, w, h, speed, type }` objects with type
  `life`, `bomb`, or `weapon`. New pickup types go in `applyPowerUp`, and
  `maybeSpawnPowerUp` decides when they drop.
- **Adaptive quality.** `trackPerf` steps a `quality` level (0–2) based on
  average frame time. Visual effects that are expensive should respect it.
- **Reduced motion.** Screen shake, hit/bomb flashes, and the low-health pulse
  must be disabled when `reducedMotion` is on. The class `reduced-motion` on
  `<body>` handles the CSS side.
- **Accessibility.** Announce important state changes through `announce()`,
  which writes to the `#srLive` live region. Keep the `title`
  attributes on icon buttons up to date.
- **Audio is opt-in by gesture.** Create the `AudioContext` lazily in
  `ensureAudio()`, which is called from user-input handlers. Do not start audio
  on page load.

## Persistence

`localStorage` keys (all prefixed `skyfighter.`):

| Key | Value | Purpose |
| --- | --- | --- |
| `skyfighter.bestScore` | number | Best score across sessions |
| `skyfighter.muted` | `'1'` / `'0'` | Sound mute |
| `skyfighter.difficulty` | `easy` / `normal` / `hard` | Selected difficulty |
| `skyfighter.reducedMotion` | `'1'` / `'0'` | Reduced-motion override (absent → follow OS preference) |

Treat storage as best-effort. If you add a key, prefix it with `skyfighter.`
and document it in the table above. Do not store anything sensitive.

## Input summary

- Keyboard: arrows or WASD move, Space fires, X or Shift bombs, P or Esc pauses.
- Mouse: move to steer, click to fire.
- Touch: drag in the left zone to move (joystick), tap FIRE, tap BOMB.

Keyboard state is tracked in a `keys` map keyed by `e.code`. Touch handlers call
`preventDefault` on the fire and bomb buttons to avoid scroll and double-fire.

## Making changes

- Keep to the single-file structure unless the user asks for a split. Do not add
  a build step, framework, or dependencies without a clear reason.
- Match the existing style: `const`/`let`, arrow functions for callbacks, named
  `function` declarations for game systems, two-space indentation, and short
  section comments using the `// ---------- Name ----------` banner.
- `renderHearts` and `renderBombs` build markup with `innerHTML` from numeric
  counts only. Never pass user-provided text into `innerHTML`; use `textContent`
  for anything else.
- Before committing, open the game in a browser and confirm there are no
  console errors and that the change works in the relevant state (menu, play,
  pause, game over, boss).

## Git

- Development branch for this work: `claude/gifted-tesla-7viy6r`.
- Commit with clear, descriptive messages. Push with `git push -u origin <branch>`.
- Do not open a pull request unless the user asks for one.
