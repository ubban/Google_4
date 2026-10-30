# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Google_4 is a static, dependency-free web project. Its only game is **Sky Fighter**, a Raiden-style vertical shoot-'em-up rendered on a single HTML5 `<canvas>`.

- `index.html` — the entire game: inline CSS (`<style>`), markup, and one IIFE-wrapped `<script>` containing all logic.
- `README.md` — a one-line title. It has no setup or usage instructions.

There is no package manager, build step, bundler, test suite, or linter. Do not add one unless asked.

## Running and verifying

- Open `index.html` directly in a browser, or serve the directory with any static server (for example `python3 -m http.server`).
- There are no automated tests. Verify changes by playing the game in a browser and checking the affected behavior, including on a narrow/touch viewport when touching layout or touch controls.
- Syntax-check the inline script after edits if you have Node available, for example by extracting the `<script>` body into a temp file and running `node --check` on it.

## Code layout inside `index.html`

The script is one `(function () { 'use strict'; ... })()` closure. Nothing is exported to `window`, so keep new code inside that closure.

Major sections, in file order (approximate line numbers shift as the file changes):

1. **DOM references** — `getElementById` lookups at the top of the closure.
2. **Persistence keys** — `STORAGE_BEST`, `STORAGE_MUTE`, `STORAGE_DIFF`, `STORAGE_REDUCED`, all prefixed `skyfighter.`.
3. **Adaptive performance** — `trackPerf` drops the `quality` level (0 full, 1 medium, 2 low) when frame time is high.
4. **Tuning tables** — `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES` (per-stage sky palettes), `ENEMY_TYPES`, `BOSSES`.
5. **Audio** — `ensureAudio`, `beep`, `noiseBurst`, and the `EXPLODE_SFX` map. Audio is synthesized with WebAudio; there are no sound files.
6. **HUD and effects** — hearts, bombs, combo pop, stage banner, screen shake, vignette, hit flash.
7. **Input and layout** — `resize()` scales a fixed `BASE_W × BASE_H` (480 × 720) logical canvas; joystick and touch handling; keyboard handlers.
8. **Game state** — `resetGame`, `update(dt)`, spawning, power-ups, weapons (`fireWeapon`), bombs (`useBomb`), chains/score (`addScore`, `breakChain`).
9. **Bosses** — `BOSS_PATTERNS` (bullet-pattern functions), `updateBoss`, `killBoss`.
10. **Rendering** — `drawPlane`, `drawBossShip`, `drawClouds`, `draw()`.
11. **Main loop** — `loop(now)` calls `update` and `draw` each frame, clamping `dt` to 0.05 s.

### Controls

- Move: arrow keys or WASD. Touch: on-screen joystick.
- Fire: Space (hold), or the on-screen fire button.
- Bomb: X or Shift, or the on-screen bomb button.
- Pause: P or Escape.

## Conventions

- Plain ES2015+ JavaScript and CSS in one file. No frameworks, no modules, no external CDN dependencies. Keep it that way.
- Follow the existing style: 2-space indentation, `const`/`let`, arrow functions for callbacks, and short section banner comments like `// ---------- Name ----------`.
- Tuning values live in the constant tables near the top of the script. Prefer changing those over adding magic numbers inline.
- Accessibility matters here and is already built in: respect `reducedMotion` (both the stored preference and `prefers-reduced-motion`), and announce important state changes through the `srLive` live region via `announce()`.
- Persisted state goes through `localStorage` with the `skyfighter.` prefix. The existing reads and writes are not wrapped in try/catch, but storage can throw in private windows or with blocked site data, so guard any new storage access.
- Keep the game's visuals and HUD readable on both large desktop and small phone viewports. Test any HUD or layout change at both sizes.

## Git

- Develop on the branch designated for the task (currently `claude/gifted-tesla-fas2qq`). Do not push to other branches without permission.
- Commit with clear, descriptive messages that explain the gameplay or code change.
- Do not open a pull request unless explicitly asked.
