# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo contains **Sky Fighter**, a Raiden-style vertical shoot-em-up built as a
single-page HTML5 canvas game. Everything lives in one file:

- `index.html` (~1670 lines): inline `<style>`, markup, and one inline `<script>` IIFE.
- `README.md`: placeholder title only.

There is no build step, no package manager, no bundler, no test suite, and no linter
configured. Open `index.html` directly in a browser to play.

## Running and verifying

- Run locally: open `index.html` in a browser, or serve the directory with any static
  server (for example `python3 -m http.server` from the repo root).
- No automated tests exist. Verify gameplay changes by playing them in a browser,
  including the touch controls when the change touches input handling.
- Check the browser console for errors after any change to the script.

## Code layout inside `index.html`

The script is wrapped in an IIFE with `'use strict'`. Keep new code inside that scope.
Sections are marked with comment banners such as `// ---------- Adaptive performance ----------`;
follow that pattern when adding a section.

Main parts, in file order:

1. **DOM references**: `const xxxEl = document.getElementById(...)` at the top.
2. **Storage keys**: `STORAGE_*` constants, all prefixed `skyfighter.`.
3. **Audio**: Web Audio helpers (`ensureAudio`, `beep`, `noiseBurst`, `panFor`).
   Sound effects are synthesized, not loaded from files.
4. **Adaptive performance**: `trackPerf` lowers or raises a `quality` level (0–2) based on
   average frame time over 45 frames.
5. **Config tables**: `DIFFICULTIES`, `THEMES`, `ENEMY_TYPES`, `BOSSES`, `BOSS_PATTERNS`,
   `KILLS_PER_STAGE`. Prefer adding data here over hard-coding values in logic.
6. **State**: persisted values (`bestScore`, `muted`, `difficulty`, `reducedMotion`) are
   loaded from `localStorage` at startup. Game state is plain `let` variables.
7. **UI helpers**: HUD rendering (`renderHearts`, `renderBombs`), feedback effects
   (`flashHit`, `screenShake`, `showComboPop`, `showStageBanner`), and `announce` for
   screen-reader live-region updates.
8. **Input**: keyboard (`keys[e.code]`), touch joystick and buttons, and pointer events.
9. **Gameplay**: `resetGame`, `update(dt)`, weapons (`fireWeapon`), bombs (`useBomb`),
   power-ups (`applyPowerUp`), enemy spawning, boss attack patterns (`fire*` helpers),
   scoring and kill-chain multiplier (`addScore`, `breakChain`), `loseLife`, `endGame`.
10. **Rendering**: `draw()` and the sprite helpers `drawPlane` and `drawBossShip`.
11. **Main loop**: `loop(now)` uses `requestAnimationFrame` and clamps `dt` to 0.05 s.
    Start and pause are handled by `startBtn`, `togglePause`, and the `P`/`Esc` keys.

## Conventions

- Use `const` by default and `let` only for reassigned state. Match the existing
  two-space indentation, single quotes, and arrow-function style.
- Keep the game logic time-based: multiply movement and timers by `dt` so behaviour
  does not depend on frame rate.
- Respect the `reducedMotion` setting when adding screen shake, flashes, or other
  motion effects. Also check `quality` before adding expensive draw work.
- Any new user-facing state change should be announced through `announce()` so
  screen readers receive it.
- Persist new settings under a `skyfighter.` key and wrap nothing in extra frameworks;
  the project is intentionally dependency-free.
- Keep the page working on touch devices: any new control needs a touch path as well as
  a keyboard path, and pointer/touch handlers call `preventDefault` where needed.

## Controls (for reference when testing)

- Move: arrow keys or WASD (keyboard), joystick (touch).
- Fire: Space (keyboard), fire button (touch).
- Bomb: `X` or Shift (keyboard), bomb button (touch).
- Pause: `P` or `Esc`.
- Mute and motion toggles are in the HUD.

## Git workflow

- Develop on the branch assigned for the session (currently `claude/gifted-tesla-x95f8r`).
- Commit with clear, descriptive messages and push with `git push -u origin <branch>`.
- Do not open a pull request unless explicitly asked.
