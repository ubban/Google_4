# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a small, dependency-free repo containing a single browser game, **Sky Fighter** — a Raiden-style vertical shoot-'em-up rendered on an HTML5 `<canvas>`.

Files:

- `index.html` (~1670 lines) — the entire game: inline `<style>` (lines ~10–359), markup (~361–421), and one inline `<script>` IIFE (~422–1667).
- `README.md` — a stub; no setup or usage docs yet.

There is no `package.json`, build step, bundler, test suite, linter, or CI configuration. Do not invent npm/yarn commands for this repo.

## Running the game

Open `index.html` directly in a browser, or serve the directory with any static server (e.g. `python3 -m http.server`). No install step is required.

For verification of visual or gameplay changes, load the page in a real browser (Playwright/Chromium is preinstalled in the cloud environment) rather than relying on reading the code alone.

## Architecture of `index.html`

Everything lives in one IIFE; there are no modules, globals, or external scripts.

- **Constants & config (top of script):** `BASE_W = 480`, `BASE_H = 720` (logical canvas size, scaled by `resize()`), `KILLS_PER_STAGE = 18`, `DIFFICULTIES` (`easy` / `normal` / `hard` — spawn rate, enemy speed, shot chance, lives, bombs, boss HP multiplier), `THEMES` (six per-stage sky palettes), `ENEMY_TYPES` (`grunt`, `interceptor`, `tank`), and `BOSSES` (six bosses cycled by stage, each with its own HP multiplier and colors).
- **Boss attack patterns:** `BOSS_PATTERNS` plus the `fire*` helpers (radial, aimed spread, columns, spiral, cross, homing, double ring). New patterns should be added as entries there.
- **Game loop:** `update(dt)` handles simulation (movement, spawning, collisions, power-ups, boss logic); `draw()` renders; `loop(now)` drives both with a delta-time step. Keep simulation and rendering separated this way.
- **Adaptive quality:** `trackPerf()` samples frame times and steps a `quality` level (0 full / 1 medium / 2 low) to reduce effects when frame rate drops. Respect this when adding expensive effects.
- **Audio:** WebAudio synthesis only (`beep`, `noiseBurst`, `ensureAudio`) — there are no audio asset files. Audio must be started from a user gesture (`ensureAudio()`).
- **Input:** keyboard (`keys[e.code]`; arrows/WASD move, Space fires, X/Shift bomb, P/Esc pause), mouse (move to steer, click to fire), and touch (left-side joystick zone, FIRE and BOMB buttons). Preserve all three when changing controls.
- **Persistence:** `localStorage` only, under keys prefixed `skyfighter.` — `bestScore`, `muted`, `difficulty`, `reducedMotion`. Every access should tolerate storage being unavailable.
- **Accessibility:** `announce()` writes to the `aria-live` region `#srLive`; a reduced-motion mode (`body.reduced-motion`) disables screen shake and similar effects, defaulting to the OS `prefers-reduced-motion` setting.

### Gameplay systems to keep consistent

- Weapons: `vulcan` and `laser` (toggled by a weapon power-up), each with power levels; fire delay differs by weapon.
- Power-ups: `life`, `bomb`, `weapon`. Colors are set in `draw()`.
- Bombs: limited per run (from difficulty); clear enemy bullets and enemies on screen and deal 24 damage to the active boss.
- Score: kill-chain multiplier (`addScore`, `breakChain`) on top of enemy base scores.
- Stages: reaching `KILLS_PER_STAGE` kills spawns the stage boss; defeating it advances to the next stage. The stage number picks the theme (`THEMES`) and boss (`BOSSES`), both cycling with modulo.

## Conventions

- Keep the game as a single self-contained `index.html` unless the user asks for a restructure. Do not split files or add a build system without being asked.
- Stay with vanilla JavaScript and Canvas 2D. Do not add frameworks or CDN dependencies without explicit approval.
- Match the existing style: 2-space indentation, `const`/`let`, arrow functions in callbacks, section comments with `// ---------- Name ----------`, and CSS custom values written inline.
- When adding a gameplay value, put it in the relevant config object (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, …) rather than hard-coding it in the loop.
- Any new user-facing text should go through the existing HUD/overlay elements and, where it conveys state, through `announce()`.
- Verify changes in a browser, including the touch layout at phone width (the canvas scales to the viewport; `touch-action: none` is set on the body).

## Git

- Development branch for this workspace: `claude/gifted-tesla-y6s0oi`. Commit and push there; do not push to other branches without explicit permission.
- Do not open a pull request unless the user explicitly asks for one.
- Commit messages should be short and imperative, with a body for anything non-trivial (the existing history follows this style, e.g. "Add Sky Fighter: Raiden-style vertical shoot-em-up game").

## Known gaps

- `README.md` is a placeholder. If you are asked to document the project, replace it with a description of the game and how to run it, rather than adding a second doc.
- No automated tests exist. Any test harness added later should be documented here, including the exact command to run it.
