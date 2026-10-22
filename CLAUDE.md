# CLAUDE.md

Guidance for working in this repository.

## Project overview

Google_4 is a small static web project. Its only real content is **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs in the browser on an HTML5 canvas.

- `index.html` — the entire game: markup, CSS (`<style>`, ~lines 10–359), canvas element, and JavaScript (`<script>`, starting ~line 422).
- `README.md` — minimal project title only.

There is no build step, no package manager, no dependencies, no test suite, and no linter configured. Do not add any of these unless asked.

## Running the game

Open `index.html` directly in a browser, or serve the directory with any static server (e.g. `python3 -m http.server`). No install is needed.

## Code architecture (`index.html`)

All JavaScript lives in a single IIFE with `'use strict'`. Nothing is exported to `window`; keep it that way.

Major sections, in file order:

1. **DOM references** — `const` lookups by element id at the top of the IIFE. Any new HUD element needs its id looked up here.
2. **Persistence keys** — `STORAGE_*` constants (`skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion`). Reads happen once at startup; writes happen on change.
3. **Tuning tables** — the places to change game balance:
   - `DIFFICULTIES` — spawn rate, enemy speed, enemy fire chance, starting lives/bombs, boss HP multiplier (`easy` / `normal` / `hard`).
   - `KILLS_PER_STAGE` — kills needed to advance a stage.
   - `THEMES` — per-stage sky gradient, cloud and sun colors.
   - `ENEMY_TYPES` — `grunt`, `interceptor`, `tank`: size, HP, speed multiplier, score, `minStage`.
   - `BOSSES` — boss names, colors, and HP multiplier, cycled per boss.
   - `BOSS_PATTERNS` — bullet-pattern functions (`fireRadial`, `fireAimedSpread`, `fireCross`, `fireSpiralBurst`, `fireHomingSeeded`, `fireDoubleRing`, etc.) used by `updateBoss`.
4. **Audio** — `ensureAudio()` lazily creates the `AudioContext`; `beep()` and `noiseBurst()` synthesize all sound effects. There are no audio files. `sfx` holds the named effects; `EXPLODE_SFX` maps enemy types to explosion sounds.
5. **UI feedback** — `renderHearts`, `renderBombs`, `flashHit`, `flashBomb`, `showComboPop`, `showStageBanner`, `screenShake`, `announce` (screen-reader live region).
6. **Layout / input** — `resize()` scales a fixed `BASE_W × BASE_H` (480×720) logical canvas to the viewport. Touch joystick (`joystick*`), fire and bomb buttons, and mouse drag are handled here, along with keyboard listeners.
7. **Game state** — `resetGame()` initializes everything. Entities are plain arrays/objects (enemies, bullets, power-ups, explosions) updated each frame.
8. **Simulation** — `update(dt)` runs movement, firing (`fireWeapon`), collisions (`rectsOverlap`), scoring (`addScore`, combo chain via `breakChain`), bombs (`useBomb`), boss logic (`spawnBoss`, `updateBoss`, `killBoss`), life loss (`loseLife`), and game over (`endGame`).
9. **Rendering** — `draw()` renders in order: background/clouds (`drawClouds`), enemies (`drawPlane`, `drawBossShip`), bullets, power-ups, particles, HUD on canvas. Shake is applied via `shakeTime`.
10. **Main loop** — `loop(now)` clamps `dt` to 50 ms, calls `trackPerf` (adaptive `quality` level 0–2), then `update` + `draw` when running and not paused. It is driven by `requestAnimationFrame`.

### Conventions

- Time-based movement: multiply by `dt` (seconds). Do not assume a fixed frame rate.
- Coordinates are in logical `BASE_W × BASE_H` units; do not use raw CSS pixels in game logic.
- Respect `reducedMotion` for any new screen shake, flashes, or large motion effects.
- Keep sound on the existing `beep` / `noiseBurst` helpers rather than introducing audio assets.
- Keep UI state changes accessible: announce important events through `announce()`.
- Match the existing style: 2-space indent, single quotes, `const`/`let`, arrow functions for callbacks, and short section-level comments only where the code is not self-explanatory.

## Controls (for reference when testing)

| Action | Keyboard | Touch / mouse |
| --- | --- | --- |
| Move | Arrows / WASD | Left joystick / drag on canvas |
| Fire | Space (hold) | Fire button (hold) |
| Bomb | X or Shift | Bomb button |
| Pause | P or Esc | Pause button |
| Mute | — | Mute button (persisted) |

## Testing changes

No automated tests exist. Verify gameplay changes by loading `index.html` in a browser and playing through:

- Start, pause/resume, and game over (new-best banner when applicable).
- Each difficulty level.
- At least one stage transition and one boss fight.
- Power-up pickup, bomb use, and life loss.
- A narrow (phone-sized) viewport with touch controls visible.

Check the browser console for errors after each run.

## Git workflow

- Develop on the branch assigned for the session; do not push to other branches without permission.
- Commit with clear, descriptive messages that explain the gameplay or UI change.
- Keep commits focused: balance tweaks, visual changes, and refactors should be separate commits where practical.
