# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Google_4 is a small, dependency-free static web project. Its only real content is **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs in the browser on a `<canvas>`.

- `index.html` — the entire game: markup, CSS (inline `<style>`), and JavaScript (one inline `<script>`, wrapped in an IIFE with `'use strict'`).
- `README.md` — a placeholder title line. It has no setup or usage instructions yet.

There is no package manager, build step, bundler, test suite, linter, or CI configuration in the repo.

## Running and verifying

- To run the game, open `index.html` directly in a browser, or serve the directory with any static server (for example `python3 -m http.server` from the repo root, then visit `http://localhost:8000`).
- There are no automated tests. Verify changes by playing the game in a browser. For gameplay or layout changes, exercise the affected path (start, fire, bomb, pause, stage change, boss, game over) and check the browser console for errors.
- For touch-related changes, test with touch emulation, since touch controls only appear on touch-capable devices (`isTouchDevice` in `index.html`).

## Code layout inside `index.html`

The script is organized into commented sections. Keep new code in the matching section:

| Section (comment) | Contents |
| --- | --- |
| DOM lookups | `getElementById` / `querySelectorAll` constants at the top of the IIFE |
| Storage keys | `STORAGE_*` constants for `localStorage` keys (all prefixed `skyfighter.`) |
| Adaptive performance | `quality` (0 full / 1 medium / 2 low) adjusted from average frame time |
| Config tables | `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES` (per-stage sky palettes), `ENEMY_TYPES`, `BOSSES` |
| Audio | Web Audio synth (`beep`, `noiseBurst`, `sfx`); created lazily by `ensureAudio()` |
| HUD / feedback | hearts, bombs, combo pop, stage banner, screen shake, flashes |
| Layout / touch | `resize()` scales a `480×720` base canvas to fit; joystick and button handling |
| Game state | `resetGame()`, `update(dt)`, spawning, collisions, scoring, bombs, weapons, power-ups |
| Boss patterns | `BOSS_PATTERNS` and the `fire*` helpers that build bullet patterns |
| Drawing | `draw()` and the `draw*` helpers |
| Loop | `loop(now)` drives `update` and `draw` via `requestAnimationFrame`; `dt` is clamped to 0.05 s |

Notes on the existing design:

- **Game loop:** `update(dt)` mutates state; `draw()` only renders. Keep that separation.
- **Input:** Keyboard state is tracked in a `keys` map keyed by `KeyboardEvent.code`. Movement uses arrows or WASD; fire is Space; bomb is X or Shift; pause is P or Escape. Touch controls are a virtual joystick plus Fire and Bomb buttons.
- **Persistence:** Only the best score, mute flag, difficulty, and reduced-motion preference are saved, in `localStorage`. Reads and writes are not wrapped in try/catch today, so be careful adding more storage calls, since some browsers throw when storage is blocked.
- **Accessibility:** The `announce()` helper writes to an ARIA live region (`#srLive`). Reduced motion is honored from `prefers-reduced-motion` and can be toggled by the player.
- **Difficulty:** Lives, bombs, spawn rates, enemy speed, shoot chance, and boss HP multiplier all come from `DIFFICULTIES`. Prefer adding tuning values there rather than hard-coding them.

## Conventions

- Keep the project dependency-free and single-file unless the user asks otherwise. Do not introduce a build system, package manager, or external CDN scripts without asking.
- Match the existing style in `index.html`: 2-space indentation, single quotes in JavaScript, `const` by default, arrow functions for callbacks, and short section-divider comments of the form `// ---------- Name ----------`.
- Put tunable numbers in the config tables near the top of the script, not inline in update or draw logic.
- Preserve the IIFE and `'use strict'` wrapper. Do not add globals to `window`.
- Do not remove the `aria-live` announcements or the reduced-motion path when changing feedback effects.

## Git workflow

- Develop on the branch assigned for the session. Do not push to other branches without explicit permission.
- Commit only when asked. Keep commits focused on one change.
- Do not open a pull request unless the user explicitly asks for one.

## Things to avoid

- Do not commit secrets, `.env` files, or credentials.
- Do not add generated or vendored files to the repository.
- Do not change the storage key names (`skyfighter.*`). Renaming them would silently reset players' saved best score, mute, difficulty, and motion settings.
