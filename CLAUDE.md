# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo contains **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs in the browser. The whole game is a single self-contained file:

- `index.html` — HTML markup, inline `<style>`, and one inline `<script>` (an IIFE, `'use strict'`) holding all game logic. Roughly 1,670 lines.
- `README.md` — placeholder; contains no project documentation yet.

There is no build step, no package manager (no `package.json`), no bundler, no test suite, and no linter configured. Do not add tooling unless asked.

## Running the game

Open `index.html` directly in a browser, or serve the directory with any static server, for example:

```bash
python3 -m http.server 8000   # then visit http://localhost:8000/index.html
```

## Code layout inside `index.html`

The script is organized into these sections, roughly in order:

1. **DOM references** — `getElementById` lookups for HUD, overlay, and touch controls.
2. **Config constants** — `DIFFICULTIES` (easy/normal/hard tuning: spawn rates, enemy speed, shoot chance, lives, bombs, boss HP multiplier), `KILLS_PER_STAGE`, `THEMES` (per-stage sky palettes), `ENEMY_TYPES`, `BOSSES`, `BOSS_PATTERNS`.
3. **Persistence** — `localStorage` keys prefixed `skyfighter.` (`bestScore`, `muted`, `difficulty`, `reducedMotion`). Every read and write is a direct `localStorage` call; keep that consistent if adding settings.
4. **Audio** — Web Audio helpers (`ensureAudio`, `beep`, `noiseBurst`). Audio is created lazily on the first user gesture.
5. **Input** — `keys` map keyed by `KeyboardEvent.code`, a keydown/keyup listener pair, and touch joystick/fire/bomb handling.
6. **Game state & entities** — `resetGame`, `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`, `spawnExplosion`, `update(dt)`.
7. **Weapons & boss patterns** — `fireWeapon`, `fireRadial` / `fireAimedSpread` / `fireSpiralBurst` / etc., and the `BOSS_PATTERNS` table driven by `updateBoss`.
8. **Rendering** — `drawPlane`, `drawBossShip`, `drawClouds`, `draw()`. Drawing is done on a single 2D canvas (`#gameCanvas`).
9. **Main loop** — `loop(now)` uses `requestAnimationFrame`, clamps `dt` to 0.05 s, and calls `trackPerf` for adaptive performance tuning.

### Controls (as implemented)

| Action | Keyboard | Touch |
| --- | --- | --- |
| Move | Arrow keys or WASD | Left-side joystick |
| Fire (hold) | Space | Fire button (touch and mouse) |
| Smart bomb | X or Shift | Bomb button |
| Pause | P or Escape | Pause button (click) |
| Mute | No key binding | Mute button (click) |

The game renders at a fixed logical size (`BASE_W = 480`, `BASE_H = 720`) and `resize()` scales the canvas to fit the viewport.

## Conventions

- **Keep it a single file.** New game logic, styles, and markup belong in `index.html` unless the user asks for a split.
- **Stay in the IIFE.** Do not leak globals; use `const`/`let` and keep the `'use strict'` wrapper.
- **Tune through the config tables.** Difficulty and boss numbers live in `DIFFICULTIES`, `BOSSES`, and `BOSS_PATTERNS`; prefer adjusting those over hard-coding magic numbers in `update`/`draw`.
- **Respect reduced motion and mute.** New visual effects such as screen shake or flashes should honor the `reducedMotion` setting; new sounds should go through the existing `beep`/`noiseBurst` helpers so the mute toggle applies.
- **Keep touch working.** Any new control needs a touch equivalent, and `touch-action: none` / `user-select: none` on the body should stay in place.
- **Accessibility hooks.** Status changes are announced via the `srLive` live region (`announce()`); reuse it for new player-facing messages.

## Verifying changes

Since there is no automated test setup, verify by running the game in a browser:

- Start a run, and check movement, firing, bombs, power-ups, stage transitions, and boss fights.
- Check each difficulty level and that the best score persists across reloads.
- Confirm the layout still works at phone width and with touch controls.
- Confirm mute and reduced-motion toggles still take effect.

A headless Playwright run (Chromium is pre-installed in the remote environment) can be used for smoke-testing: load the page, click `#startBtn`, and check for console errors.

## Git workflow

- Develop on the branch assigned to the session (currently `claude/gifted-tesla-xie0xd`).
- Do not open a pull request unless the user asks for one.
- Keep commit messages short and focused on the player-visible or code-level change.
