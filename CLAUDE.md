# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo contains **Sky Fighter**, a Raiden-style vertical shoot-em-up that runs in a single browser page. There is no build step, no package manager, and no test suite.

Files:

- `index.html` — the entire game: markup (~lines 1–360), CSS (`<style>`, ~lines 10–360), and one inline `<script>` (~lines 422–1667) wrapped in an IIFE with `'use strict'`.
- `README.md` — minimal placeholder.

## Running it

Open `index.html` directly in a browser, or serve the folder with any static server (e.g. `python3 -m http.server`). Nothing needs to be installed.

## Architecture (inside `index.html`)

- **Rendering:** a single `<canvas id="gameCanvas">` drawn with the 2D context. The logical play area is `BASE_W × BASE_H` = 480 × 720; `resize()` scales it to the viewport. Keep game coordinates in the logical space, not CSS pixels.
- **Main loop:** `loop(now)` is driven by `requestAnimationFrame`, clamps `dt` to 0.05 s, calls `update(dt)` then `draw()` only while `running && !paused`.
- **State:** plain module-level `let` variables inside the IIFE (player, enemies, bullets, score, combo, weapon level, boss state, etc.). `resetGame()` reinitializes them.
- **Tuning data:** constants near the top of the script — `DIFFICULTIES` (spawn rates, enemy speed, shoot chance, lives, bombs, boss HP multiplier), `KILLS_PER_STAGE`, `THEMES`, `ENEMY_TYPES`, `BOSSES`, and `BOSS_PATTERNS` (bullet pattern functions such as `fireRadial`, `fireCross`, `fireHomingSeeded`). Prefer changing these tables over hard-coding numbers in logic.
- **Audio:** Web Audio synthesized in code (`beep`, `noiseBurst`, `EXPLODE_SFX`). `ensureAudio()` must run from a user gesture before sound plays.
- **Adaptive quality:** `trackPerf()` watches frame time over a 45-frame window and raises/lowers a `quality` level to keep the game smooth on slow devices.
- **Persistence:** `localStorage` keys are prefixed `skyfighter.` — `bestScore`, `muted`, `difficulty`, `reducedMotion`. Every access must tolerate missing or malformed values (they are read with `Number(...) || 0` and string comparisons today; keep that defensive style).
- **Accessibility:** a live region (`srLive`, via `announce()`) announces game events; a reduced-motion toggle gates screen shake and flashes.

## Controls (for reference when editing input code)

- Keyboard: arrow keys or WASD for movement (tracked in `keys[e.code]`); `Space` fires (default prevented); `P` or `Escape` pauses; `X` or either `Shift` drops a bomb.
- Touch: a virtual joystick on `moveZone`, a fire button, and a bomb button. Touch controls are shown only when `isTouchDevice` is true.
- Pointer: `canvasPointFromEvent()` converts client coordinates into logical canvas coordinates.

## Conventions

- Keep everything in `index.html` unless there is a clear reason to split it. The project is deliberately dependency-free; do not add a bundler, framework, or npm dependency without asking.
- Match the existing style: 2-space indentation, `const`/`let`, arrow functions for callbacks, short section comments like `// ---------- Name ----------`.
- Use `function` declarations for game logic helpers, as the file already does.
- Do not introduce `console.log` debugging left in committed code.
- `innerHTML` is used only in `renderHearts()` and `renderBombs()`, and only with numeric counts. Don't assign `innerHTML` from strings that could contain user or stored data; use `textContent` for text.
- Keep the page usable at phone width and with touch input; test any input change on both keyboard and touch paths.

## Verifying changes

There are no automated tests or linters. Before reporting a change as working:

1. Open `index.html` in a browser (or serve it locally) and check the browser console for errors.
2. Start a game, play through at least one stage transition and a boss fight if the change touches those paths.
3. Check each difficulty level if you changed `DIFFICULTIES`.
4. For gameplay or layout changes on small screens, check a touch-sized viewport as well.

If you can't run the game in a browser, say so explicitly rather than claiming it works.

## Git

- Default development branch for this work: `claude/gifted-tesla-gvdlk9`.
- Commit messages are short imperative summaries (the history uses messages like "Add Sky Fighter: ...").
- Do not open pull requests unless explicitly asked.
