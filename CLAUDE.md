# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo contains **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs as a single-page HTML5 canvas game.

- `index.html` — the entire game: inline `<style>` (lines ~10–360), the DOM/HUD/overlay markup (~361–421), and one inline `<script>` IIFE (~422–end) that holds all game logic.
- `README.md` — a placeholder with no setup instructions.

There is no build step, no package manager, no bundler, no test suite, and no linter configuration. Opening `index.html` in a browser is the way to run it.

## Running and verifying

- Run: open `index.html` directly in a browser, or serve the directory with any static server (e.g. `python3 -m http.server`) and visit it.
- There are no automated tests. Verify changes by playing the game in a browser, ideally on both desktop (keyboard/mouse) and a touch-sized viewport (touch controls).
- Keep the file a single self-contained HTML document unless the user asks for a split. Do not add external dependencies or CDN links without a reason.

## Code structure (inside the `<script>` IIFE)

Roughly in order of appearance:

1. **DOM references** — `document.getElementById(...)` lookups at the top of the IIFE. If you add a new HUD or overlay element, add its reference here.
2. **Storage keys** — `STORAGE_BEST`, `STORAGE_MUTE`, `STORAGE_DIFF`, `STORAGE_REDUCED`, all prefixed `skyfighter.`. These are the only persisted state.
3. **Adaptive performance** — `trackPerf` moves a `quality` level (0 full / 1 medium / 2 low) based on average frame time.
4. **Tuning tables** — `DIFFICULTIES` (spawn rates, enemy speed, shoot chance, lives, bombs, boss HP multiplier), `KILLS_PER_STAGE`, `THEMES` (per-stage sky/cloud/sun palettes), `ENEMY_TYPES` (grunt, interceptor, tank), `BOSSES` (six boss designs, cycled by stage). Balance changes usually belong here.
5. **Audio** — `ensureAudio`, `beep`, `noiseBurst`, and the `EXPLODE_SFX` map. Audio is created lazily on the first user gesture and is wrapped in try/catch.
6. **HUD helpers** — hearts/bombs rendering, hit flash, combo pop, stage banner, screen shake.
7. **Input** — keyboard (`keys[e.code]`), touch joystick and fire/bomb buttons, and mouse/pointer on the canvas.
8. **Game state** — `resetGame()`, `update(dt)` (the main simulation step), `applyPowerUp`, `useBomb`, `fireWeapon`, `loseLife`, `endGame`, `togglePause`.
9. **Bosses** — `BOSS_PATTERNS` is an array of bullet-pattern functions (`fireRadial`, `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`, `fireCross`, `fireHomingSeeded`, `fireDoubleRing`). `updateBoss` and `killBoss` drive the fight.
10. **Rendering** — `drawPlane`, `drawBossShip`, `drawClouds`, `draw()`.
11. **Main loop** — `loop(now)` clamps `dt` to 50 ms, runs `update` and `draw` only while `running && !paused`, and starts from the `startBtn` click handler.

## Conventions

- Plain JavaScript (ES2015+), no modules, no TypeScript, no transpilation. Keep the existing `'use strict'` IIFE and do not leak globals.
- Game coordinates use a fixed logical canvas of `BASE_W = 480` × `BASE_H = 720`; `resize()` scales it to the viewport. Write positions and sizes in logical units.
- Use `const` by default; `let` only for values that change. Match the existing 2-space indentation and single quotes.
- Naming: `UPPER_SNAKE` for tuning tables and constants, `camelCase` for functions and variables.
- Per-frame code should avoid allocations where practical. `quality` exists so effects can be reduced on slow devices; respect it when adding particles or expensive draws.
- Any new user-facing text that changes game state should go through `announce()` for screen readers. Motion-heavy effects (shake, flash, pulse) must honor `reducedMotion` (`body.reduced-motion` class).
- Persist only through the `skyfighter.*` keys above. Keep new keys namespaced the same way.

## Known gotchas

- `localStorage` reads and writes at the top of the IIFE and in `setDifficulty`/mute/motion handlers are **not** wrapped in try/catch. In private browsing or with blocked storage they can throw. If you touch that code, wrap the access rather than assuming it works.
- The main loop is driven by `requestAnimationFrame`; `dt` is clamped, so long pauses (tab switch) do not cause large simulation jumps. Do not remove that clamp.
- Only `Space` calls `preventDefault` on keydown (to stop page scrolling). Arrow keys are not prevented, so if you bind them to something that scrolls, handle that explicitly.
- Keyboard controls: move with Arrows or WASD (`ArrowLeft`/`KeyA` etc.), fire with Space, bomb with `KeyX` or Shift, pause with `KeyP` or `Escape`. Keep the `controlsHint` text in the overlay in sync when these change.
- Bosses: `BOSSES[(stage - 1) % BOSSES.length]` picks the design and `BOSS_PATTERNS[designIndex]` picks the attack set, so the boss list cycles after six stages.

## Git workflow

- Develop on the branch the task specifies (currently `claude/gifted-tesla-j1smr7`). Do not push to other branches without permission.
- Commit with clear, descriptive messages. Do not open a pull request unless explicitly asked.
- `git push -u origin <branch>` is the push command; retry only on network failures.

## Scope for future changes

When asked to add features, prefer small, self-contained edits inside the existing sections above. If a change needs a new section (e.g. a second level type, a new screen), add it in the same style and keep the section order sensible. If the game grows past what a single file handles comfortably, propose splitting it before doing so.
