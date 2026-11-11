# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo contains **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs as a single self-contained HTML5 canvas page.

- `index.html` — the entire game: markup (~lines 1–420), CSS (inline `<style>`, ~lines 10–362), and JavaScript (inline `<script>`, ~lines 422–end).
- `README.md` — minimal placeholder.

There is no build step, no package manager, no bundler, no framework, and no test suite. Do not add one unless asked.

## Running and verifying

- Open `index.html` directly in a browser, or serve the directory with any static server (e.g. `python3 -m http.server`).
- There are no automated tests or linters configured. Verify changes by playing the game in a browser: start a run, check movement and firing, bombs, pause, stage transitions, boss fights, and game over / best-score persistence.
- For touch behavior, test with a mobile-width viewport (the page is designed to work at phone width with on-screen joystick, fire, and bomb controls).

## Architecture (inside `index.html`)

The game script is a single IIFE with `'use strict'`. Key sections, in rough order:

1. **DOM references** — cached `getElementById` lookups at the top of the IIFE.
2. **Persistence keys** — `STORAGE_*` constants (see below).
3. **Adaptive performance** — `trackPerf` samples frame times and steps a `quality` level (0 full / 1 medium / 2 low).
4. **Config tables** — `DIFFICULTIES`, `THEMES` (per-stage sky palettes), `ENEMY_TYPES`, `BOSSES`, `KILLS_PER_STAGE`.
5. **Audio** — lazily created `AudioContext` (`ensureAudio`), with `beep`/`noiseBurst` helpers and named `sfx` sounds. Audio is created only after a user gesture.
6. **Visual feedback helpers** — hearts/bombs HUD rendering, hit flash, combo popups, stage banner, screen shake.
7. **Input** — keyboard (`keydown`/`keyup` into a `keys` map keyed by `KeyboardEvent.code`), pointer, and touch joystick/buttons.
8. **Game state** — `resetGame`, `update(dt)` (movement, firing, collisions, spawning, scoring), `updateBoss`, `killBoss`, `loseLife`, `endGame`, `togglePause`.
9. **Weapons and bosses** — `fireWeapon`, `applyPowerUp`, and a `BOSS_PATTERNS` array of bullet-pattern functions (radial, aimed spread, columns, spiral, cross, homing, double ring).
10. **Rendering** — `draw()` paints clouds, planes, bosses, bullets, and effects on the canvas each frame.
11. **Main loop** — `loop(now)` drives `update` and `draw` via `requestAnimationFrame`.

Game dimensions use a fixed logical resolution (`BASE_W = 480`, `BASE_H = 720`) scaled by `resize()`. Keep gameplay coordinates in that logical space.

## Controls

| Action | Keyboard | Touch |
| --- | --- | --- |
| Move | Arrow keys or WASD | Left-side joystick |
| Fire | Space (hold) | Fire button |
| Smart bomb | X or Shift | Bomb button |
| Pause | P or Escape | Pause button |
| Mute | Mute button | Mute button |

## Persistence

The game stores state in `localStorage` under these keys. Changing a key name or value format is a breaking change for existing players' saved data, so do it deliberately:

- `skyfighter.bestScore` — numeric best score
- `skyfighter.muted` — `'1'` or `'0'`
- `skyfighter.difficulty` — `'easy' | 'normal' | 'hard'`
- `skyfighter.reducedMotion` — `'1'` or `'0'`; when absent, falls back to `prefers-reduced-motion`

All `localStorage` access should tolerate being unavailable (e.g. private browsing) without breaking the game.

## Gameplay tuning

Balance lives in the config tables near the top of the script, so prefer editing those over hard-coding numbers in logic:

- `DIFFICULTIES` — spawn rates, enemy speed and fire chance, starting lives and bombs, boss HP multiplier.
- `ENEMY_TYPES` — size, HP, speed multiplier, score value, and the minimum stage each enemy appears in.
- `BOSSES` — names, colors, and HP multiplier per boss. Boss selection is `BOSSES[(stage - 1) % BOSSES.length]`, and boss HP is `(70 + stage * 35) × difficulty.bossHpMul × design.hpMul`.
- `KILLS_PER_STAGE` — kills required to trigger a boss fight.

## Conventions

- Keep everything in `index.html` unless there is a clear reason to split files; the single-file, no-build design is intentional.
- Do not add external dependencies or CDN script tags without a specific request.
- Preserve accessibility hooks: the `srLive` live region is used by `announce()`, and reduced-motion support should continue to gate screen shake and other motion effects.
- Keep the mobile layout working: the page disables user zoom and uses `viewport-fit=cover`, and touch controls must remain usable at phone width.
- Match the existing style: 2-space indentation, single quotes in JS, `const` by default, and short helper functions grouped by section.

## Git workflow

- Develop on the branch specified for the task (currently `claude/gifted-tesla-ny1lpj`).
- Write clear commit messages that describe the gameplay or code change.
- Do not open pull requests unless explicitly asked.
