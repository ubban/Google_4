# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a small, dependency-free static web project. Its only real content is **Sky Fighter**, a Raiden-style vertical shoot-em-up built as a single-page HTML5 canvas game.

- `index.html` — the entire game: markup, CSS (`<style>` block), and JavaScript (`<script>` block, starting around line 422). ~1,670 lines.
- `README.md` — minimal placeholder (just the repo name).

There is no build step, no package manager, no bundler, and no test suite.

## Running the game

Open `index.html` directly in a browser, or serve the directory with any static server, for example:

```bash
python3 -m http.server 8000   # then visit http://localhost:8000
```

## Architecture notes (`index.html`)

The script is one IIFE-style block that owns all state as top-level `let`/`const` variables inside it.

- **Canvas loop:** `loop(now)` drives `update(dt)` and `draw()` via `requestAnimationFrame`. Pause is handled by `togglePause()`.
- **Game entities:** enemies are defined in `ENEMY_TYPES` (grunt, interceptor, tank); bosses in `BOSSES` (spawned by `spawnBoss()`, updated by `updateBoss(dt)`, HP bar in the `bossBarWrap` DOM element).
- **Stages:** `KILLS_PER_STAGE` kills advance the stage; `THEMES` cycles the sky palette per stage.
- **Difficulty:** `DIFFICULTIES` holds spawn rates, enemy speed, shoot chance, starting lives/bombs, and boss HP multiplier. Selected via `.diffBtn` buttons.
- **Weapons and combat:** `fireWeapon()` (Vulcan/Laser, 3 power levels), `useBomb()` (screen clear that also damages bosses), `spawnExplosion()`.
- **Adaptive quality:** `quality` (0 full / 1 medium / 2 low) is stepped up or down from averaged frame times.
- **Rendering helpers:** `drawPlane`, `drawBossShip`, `drawClouds` (parallax layers).
- **HUD:** DOM elements (score, multiplier, hearts, bombs, weapon name/level, combo popup, stage banner, vignette, flash) are updated from JS; they are not drawn on the canvas.

### Persistence

`localStorage` keys (all prefixed `skyfighter.`):

| Key | Purpose |
| --- | --- |
| `skyfighter.bestScore` | High score |
| `skyfighter.muted` | Audio mute (`'1'` / `'0'`) |
| `skyfighter.difficulty` | Selected difficulty |
| `skyfighter.reducedMotion` | Reduced-motion override; defaults to the `prefers-reduced-motion` media query when unset |

Always wrap new `localStorage` access in a guard if you add it; the existing reads do not handle a throwing or unavailable storage.

### Controls

- **Keyboard:** Arrow keys / WASD move (read from the `keys` map, keyed by `e.code`); Space fires; `X` or Shift bombs; `P` or Escape pauses.
- **Touch:** on touch devices the `touchControls` layer is enabled — a floating joystick in `moveZone`, a `fireBtn`, and a `bombBtn`.
- **Mouse:** click on the canvas or the fire button to fire (`mousedown` handlers).

## Conventions

- Keep the game as a single self-contained `index.html` unless there is a clear reason to split it. Do not introduce a build system or dependencies without asking.
- Match the existing style: 2-space indentation, `const`/`let`, semicolons, and keep game tuning values in the named constant objects (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `THEMES`) rather than hard-coding them in logic.
- Preserve mobile support when touching input: use `passive: false` + `preventDefault` on touch handlers that must suppress scrolling.
- Preserve the reduced-motion and quality-scaling paths when adding visual effects (screen shake, flashes, particles).

## Verifying changes

There are no automated tests or linters configured. To verify a change:

1. Load `index.html` in a browser and start a run.
2. Exercise the path you changed on at least one difficulty.
3. For touch changes, check the touch layout (or emulate a touch device in dev tools).
4. Check the browser console for errors during play, pause, a stage transition, and a boss fight.

## Git

- Develop on the session's designated branch and do not push to other branches without explicit permission.
- Commit messages follow the existing history: a short summary line, followed by a body explaining the change.
