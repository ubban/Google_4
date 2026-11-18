# CLAUDE.md

Guidance for Claude Code working in this repository.

## Project overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up that runs in the browser as a single static file.

- `index.html` — the entire game: markup, CSS, and one inline `<script>` (an IIFE, `'use strict'`). About 1,670 lines.
- `README.md` — a stub with only the repo title.
- No build step, no package manager, no tests, no dependencies. Nothing is fetched from a CDN.

To run it, open `index.html` in a browser, or serve the directory with any static server.

## Repository layout

```
index.html   # whole game (HTML + <style> + <script>)
README.md    # title only
```

Git: the default work branch for this session is `claude/gifted-tesla-m4t0u7`. Do not push to other branches without explicit permission.

## Architecture of `index.html`

The script is one closure. Rough order within the file:

1. **DOM refs and constants** — `STORAGE_*` keys, `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES`, `ENEMY_TYPES`, `BOSSES`, `BOSS_PATTERNS` (near line 1372).
2. **State** — `bestScore`, `muted`, `difficulty`, `reducedMotion` are loaded from `localStorage`. Per-run state (`player`, `bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`, `boss`, `stage`, `kills`, `lives`, `bombCount`, and so on) is reset by `resetGame()`.
3. **Audio** — `ensureAudio()` lazily creates the `AudioContext`. `beep()` and `noiseBurst()` drive the `sfx` helpers, and `panFor()` handles stereo.
4. **Input** — keyboard (`keys[e.code]`), mouse (`canvasPointFromEvent`), and touch (`joystickStart/Move/End`, FIRE and BOMB buttons). `isTouchDevice` decides which controls show.
5. **Game logic** — `update(dt)` is the per-frame simulation. It handles spawning, collisions, power-up pickup, weapons (`fireWeapon`), bombs (`useBomb`), combo chain (`addScore`, `breakChain`), boss updates (`updateBoss`, `killBoss`), and player damage (`loseLife`, `endGame`).
6. **Boss patterns** — `fireRadial`, `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`, `fireCross`, `fireHomingSeeded`, and `fireDoubleRing`, referenced per boss via `BOSS_PATTERNS`.
7. **Rendering** — `draw()` and the helpers `drawPlane`, `drawBossShip`, `drawClouds`. Screen shake applies a random offset to the canvas context.
8. **Main loop** — `loop(now)` runs `requestAnimationFrame`, steps `update(dt)` and `draw()`, and calls `trackPerf(dt)`.

### Core game rules (keep in sync with code when changing)

- **Difficulty** (`DIFFICULTIES`): `easy`, `normal` (default), `hard`. Each sets spawn rates, enemy speed and fire chance, starting lives and bombs, and a boss HP multiplier.
- **Stages**: 18 kills clear a stage (`KILLS_PER_STAGE`), then a boss spawns. Boss design is `BOSSES[(stage - 1) % BOSSES.length]`, so there are six bosses that cycle. Each stage uses the next `THEMES` entry for its sky.
- **Enemies**: `grunt`, `interceptor` (weaves), and `tank` (4 HP, appears from stage 2).
- **Power-ups**: `life`, `bomb`, and `weapon`. Weapon pickups raise the level from 1 to 3, then switch between `vulcan` and `laser` and reset to level 1.
- **Scoring and persistence**: the best score is saved under `localStorage` key `skyfighter.bestScore`. Other keys are `skyfighter.muted`, `skyfighter.difficulty`, and `skyfighter.reducedMotion`.

### Controls (also shown in the in-game hint)

- Desktop: mouse to steer and click to fire. Arrows or WASD to move, Space to fire, X or Shift for bomb, P or Esc to pause.
- Touch: drag on the left to fly, tap FIRE, tap BOMB.

## Conventions

- Keep everything in `index.html` unless the user asks to split it. Don't add a build step, framework, or npm dependencies without asking.
- Preserve the IIFE and `'use strict'` wrapper. New top-level names should go inside it.
- Game tuning values live in the constant objects at the top of the script (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `THEMES`). Prefer editing those over hard-coding numbers in logic.
- Time-based motion uses `dt`. Keep new movement frame-rate independent.
- Accessibility features exist and should keep working: `announce()` writes to the `#srLive` region, and `reducedMotion` disables screen shake, flashes, and pulses. It defaults to the OS `prefers-reduced-motion` setting until the player toggles it. Any new visual effect that flashes or shakes should respect that flag.
- Storage access is currently unguarded (`localStorage` is called directly). If you add more, keep the same key prefix `skyfighter.`.
- Keep the layout working on phones: the HUD and touch controls use `env(safe-area-inset-*)`, and the page is `overflow: hidden`.

## Testing and verification

There is no automated test suite or linter. To verify a change:

1. Open `index.html` in a browser (or a static server) and check the console for errors.
2. Play through at least one full stage and one boss on the affected difficulty.
3. Check both desktop (keyboard and mouse) and touch-emulation controls if input or layout changed.
4. Check mute, reduced-motion, and best-score persistence after reload if those areas changed.

Browser automation with Playwright is available in this environment (`require('playwright')`, no install needed) if a scripted check is useful.

## Git workflow

- Make focused commits with clear messages that describe the gameplay or code change.
- Do not create a pull request unless the user explicitly asks for one.
- Do not commit local secrets or editor files; the repo currently has none.
