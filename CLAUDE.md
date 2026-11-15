# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a small, dependency-free static web project. Its only real content is **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs in a single HTML file.

| File | Purpose |
| --- | --- |
| `index.html` | The entire game: markup, CSS (`<style>`, ~lines 10–362), canvas/HUD/overlay markup (~363–421), and game logic in one IIFE `<script>` (~422–1669). |
| `README.md` | Minimal project readme (currently just a title and a stray timestamp line). |

There is no build step, package manager, test suite, or linter configured. Nothing is bundled or transpiled.

## Running the game

Open `index.html` directly in a browser, or serve the directory with any static server, e.g.:

```bash
python3 -m http.server 8000   # then visit http://localhost:8000/
```

Serving over HTTP is optional but matches how it would be hosted. No network requests are needed; all audio is synthesized with the Web Audio API and all art is drawn on a `<canvas>`.

## Architecture of `index.html`

- **Single file, no dependencies.** Keep it that way unless the user asks for a change in structure. Do not add npm packages, CDN scripts, or external assets without asking.
- **Rendering:** a 2D canvas at a logical size of `BASE_W × BASE_H` (480 × 720), scaled by `resize()` to fit the viewport. Game coordinates should be expressed in the base size, not in CSS pixels.
- **Main loop:** `requestAnimationFrame(loop)`. `dt` is clamped to 50 ms. Each frame runs `update(dt)` then `draw()` only when the game is running and not paused.
- **Adaptive quality:** `trackPerf()` samples frame times over 45 frames and steps a `quality` level (0–2) up or down to protect frame rate. Respect this when adding expensive effects.
- **Config tables** near the top of the script are the main tuning surface:
  - `DIFFICULTIES` — spawn rates, enemy speed and shot chance, lives, bombs, boss HP multiplier.
  - `KILLS_PER_STAGE` — kills needed to advance a stage.
  - `THEMES` — per-stage sky/cloud/sun palettes (cycled by stage).
  - `ENEMY_TYPES` — `grunt`, `interceptor` (weaves), `tank` (4 HP, unlocks at stage 2).
  - `BOSSES` — six bosses, cycled by stage, each with its own colors and HP multiplier.
  - `BOSS_PATTERNS` — bullet-pattern functions (radial, aimed spread, columns, spiral, cross, homing, double ring).
- **Persistence:** `localStorage` keys are prefixed `skyfighter.` (`bestScore`, `muted`, `difficulty`, `reducedMotion`). Always wrap new `localStorage` access in `try/catch`-safe code; the game must still run if storage is unavailable (private mode, blocked storage).
- **Audio:** created lazily via `ensureAudio()` on the first user gesture, because browsers require one. Any new sound must go through the existing `beep()` / `noiseBurst()` helpers and respect the mute flag.
- **Accessibility:** a `prefers-reduced-motion` setting (`reducedMotion`) disables screen shake and similar effects; a `srLive` live region announces key events via `announce()`. Keep both working when adding visual effects or status changes.

## Controls (for testing)

| Action | Keyboard | Touch |
| --- | --- | --- |
| Move | Arrow keys / WASD (see `keys[]` handling) | Virtual joystick on the left |
| Fire | Hold Space (see `firing` handling) | Fire button |
| Smart bomb | `X` or Shift | Bomb button |
| Pause | `P` or `Esc` | Pause button |

## Conventions

- **Keep the code style of the file:** 2-space indentation, single quotes in JS, `const`/`let`, a single IIFE with `'use strict'`. Do not introduce modules or classes unless there is a clear reason.
- **Section comments** use the `// ---------- Name ----------` form. Add new logic under an appropriate existing section or a new one with the same style.
- **Don't split the file** into multiple files or add a build step as a side effect of an unrelated change.
- **Tune with config, not magic numbers.** New balance values belong in the config tables above.
- **Keep it playable on mobile.** The layout uses `viewport-fit=cover`, safe-area insets, and `touch-action: none`. Verify touch controls still work after UI changes.

## Verifying changes

There are no automated tests. For any gameplay or rendering change:

1. Open `index.html` in a browser (or serve it locally) and check the console for errors.
2. Start a run, exercise the changed behavior, and confirm pause, bomb, and game-over/restart still work.
3. Check at least one narrow (phone-sized) viewport and the keyboard path.
4. If a visual change is involved, take a screenshot before and after when practical.

If a step cannot be run in the current environment, say so explicitly rather than claiming it was verified.

## Git and delivery

- Default working branch for this checkout: `claude/gifted-tesla-70p294`. Commit and push there (`git push -u origin <branch>`).
- Do not push to other branches without explicit permission.
- Do not open a pull request unless the user asks for one.
- Keep commits focused. Write messages that describe the gameplay or code change, not the tooling used.

## Things to avoid

- Don't commit generated artifacts, editor folders, or `node_modules`; none are expected in this repo.
- Don't replace the game with a framework or rewrite it wholesale unless asked.
- Don't remove the `localStorage` guards, reduced-motion handling, or touch controls while making unrelated edits.
