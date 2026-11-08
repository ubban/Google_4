# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a single-page browser game, **Sky Fighter**: a Raiden-style vertical shoot-'em-up. Stages scroll through themed skies, enemies come in waves, weapons can be upgraded, smart bombs clear the screen, and each stage ends in a boss fight.

There is no build system, package manager, or test suite. The whole game lives in one file.

## Repository layout

| Path | Purpose |
| --- | --- |
| `index.html` | The entire game: HTML markup, inline `<style>`, and one inline `<script>` (IIFE) containing the game loop, rendering, audio, input, and HUD. About 1,670 lines. |
| `README.md` | One-line project title. |
| `CLAUDE.md` | This file. |

## Running the game

No build step is needed. Open the file directly in a browser:

```bash
xdg-open index.html        # Linux
open index.html            # macOS
```

Or serve the directory with any static server, for example `python3 -m http.server` and then visit `http://localhost:8000`.

## Code structure (inside `index.html`)

The script is wrapped in an IIFE. Within it, these are the main areas, in source order:

- **DOM references**: `const` lookups for every HUD element, overlay, and touch control.
- **Storage keys**: `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion` (all in `localStorage`).
- **Tuning tables**:
  - `DIFFICULTIES` (`easy` / `normal` / `hard`): spawn rates, enemy speed, enemy fire chance, starting lives and bombs, boss HP multiplier.
  - `KILLS_PER_STAGE` (18): kills needed to advance a stage.
  - `THEMES`: six sky palettes, cycled per stage.
  - `ENEMY_TYPES`: `grunt`, `interceptor` (weaves), and `tank` (4 HP, appears from stage 2).
  - `BOSSES`: six bosses (VANGUARD, BEHEMOTH, SPECTER, DREADNOUGHT, WRAITH, TITAN) with per-boss HP multipliers.
- **Audio**: lazily created `AudioContext` (`ensureAudio`), with `beep` and `noiseBurst` primitives and a `sfx` table. Muted state persists.
- **Rendering and effects**: HUD updates, hearts and bombs rendering, screen shake, hit and bomb flashes, combo pop-ups, stage banner.
- **Layout**: `BASE_W = 480`, `BASE_H = 720`. `resize()` scales the canvas to fit the window, capped at 1.6x.
- **Input**: keyboard (`keys` map), mouse (steer and click to fire), and touch (virtual joystick plus FIRE and BOMB buttons).
- **Performance**: `trackPerf` samples frame times and steps a `quality` level (0 full, 1 medium, 2 low).
- **Main loop**: `requestAnimationFrame(loop)`, started at the end of the IIFE.

## Controls (from the in-game hint)

- Mouse: move to steer, click to fire.
- Keyboard: Arrow keys or WASD to move, Space to fire, X or Shift for bomb, P or Esc to pause.
- Touch: drag left side to fly, tap FIRE, tap BOMB.
- Overlay buttons: mute, reduced motion toggle, pause.

## Conventions

- **Keep it a single file.** The game is intentionally self-contained with no external dependencies. Do not add a build step, bundler, or CDN script without a clear reason.
- **Preserve the IIFE.** Game state is private to the closure. Expose nothing on `window` unless required.
- **Tune through the constant tables.** Balance changes (spawn rates, HP, scores, stage length) should go in `DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, or `KILLS_PER_STAGE` rather than being hardcoded in logic.
- **Respect reduced motion.** New visual effects should check the `reducedMotion` flag, as the existing shake, flash, and banner effects do.
- **Keep it accessible.** Announcements go through `announce()` into the live region (`#srLive`). Overlays use the `hidden` attribute.
- **Mobile first.** The layout uses `touch-action: none`, `viewport-fit=cover`, and safe-area insets. Test any HUD or control change at phone width.

## Testing and verification

There is no automated test suite. To verify a change:

1. Load `index.html` in a desktop browser and play through at least one full stage, including a boss fight.
2. Check all three difficulties (`easy`, `normal`, `hard`) if you changed tuning values.
3. Check touch controls with browser device emulation, and resize the window to confirm scaling.
4. Confirm the best score persists across reloads (`skyfighter.bestScore`).
5. Confirm the console is clean of errors.

## Git workflow

- Default branch: `main`.
- Feature work happens on dedicated `claude/...` branches. Commit with clear messages describing the gameplay or code change.
