# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Google_4 is a small, dependency-free web project. Its only game content is **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs as a single self-contained HTML5 canvas page.

- `index.html` (~1,670 lines): the entire game. HTML, CSS, and JavaScript live in one file: `<style>` at the top, markup in the body, and a single `<script>` block starting around line 422.
- `README.md`: a placeholder title only.

There is no build step, package manager, test suite, or linter configured.

## Running and testing

- Open `index.html` directly in a browser, or serve the directory with any static server (e.g. `python3 -m http.server`).
- No automated tests exist. Verify changes by playing the game in a browser: start a run, check weapons, bombs, bosses, stage transitions, pause, and game over. Check mobile layout with touch controls when touching HUD or overlay code.

## Code structure (inside `index.html`)

The script is one IIFE-style block of `const`/`let` declarations and about 50 functions. Key areas, roughly in file order:

- **Storage keys and persistence** (~line 463): `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion`. All `localStorage` access must be wrapped so the game still works when storage is blocked or unavailable.
- **Tuning tables** (~lines 485–515): `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES` (per-stage palettes), `ENEMY_TYPES`, and `BOSSES`. Balance changes usually belong here, not in the loop.
- **Audio** (~line 548): a lazily created `AudioContext`. Sound is generated in code; there are no audio asset files. Respect the mute setting.
- **Canvas sizing** (~line 753): the logical play field is `BASE_W = 480` × `BASE_H = 720`, scaled to fit the viewport.
- **Input** (~line 769): keyboard listeners (`keydown`/`keyup` on `e.code`), mouse, and on-screen touch controls (joystick, FIRE, BOMB).
- **Spawning and difficulty** (~lines 876–1202): spawn timing scales with `difficultyRamp = floor(frame / 300) * 4`, and per-difficulty values come from `DIFFICULTIES`.
- **Boss patterns** (~line 1372): `BOSS_PATTERNS` define each boss's attack behaviour.
- **Main loop** (~lines 1652–1665): `requestAnimationFrame(loop)`. Keep per-frame work inside this loop and avoid allocating inside hot paths where possible.

## Conventions

- Keep everything in `index.html` unless a change clearly needs a second file. Do not add a build system or frameworks without asking.
- Match the existing style: 2-space indentation, single quotes in JS, `const` by default, and `camelCase` for functions and variables. Constants use `UPPER_SNAKE_CASE`.
- Comments should be sparse and explain *why*, matching the density of the surrounding code.
- Keep the page usable on phones: the viewport meta disables zoom, `touch-action: none` is set on the body and canvas, and HUD elements use `env(safe-area-inset-*)`. Preserve these when editing layout.
- Accessibility features exist and should be kept working: `aria-live` announcements (`#srLive`), a reduced-motion toggle that disables shake, flash, and pulse effects, and labelled icon buttons.

## Git workflow

- Develop on the branch assigned for the session (currently `claude/gifted-tesla-yv7umj`). Do not push to other branches without explicit permission.
- Write clear commit messages that describe the gameplay or UI change.
- Do not open a pull request unless explicitly asked.
