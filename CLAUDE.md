# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo is **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs as a single self-contained HTML5 canvas page.

- `index.html` — the entire game: markup (~lines 1–420), CSS (inline `<style>`, ~lines 10–360), and JavaScript (one IIFE in an inline `<script>`, starting ~line 422).
- `README.md` — minimal placeholder; no build or setup instructions exist.

There is no package manager, bundler, test suite, linter, or CI config. Do not invent npm/build steps.

## Running and verifying

- Open `index.html` directly in a browser, or serve the folder with any static server (e.g. `python3 -m http.server`).
- There are no automated tests. Verify changes by loading the page in a browser and playing through the affected path (menu → start → gameplay → pause → game over).
- Check the browser console for errors after every JavaScript change.

## Architecture (inside `index.html`)

All game code lives in a single IIFE. Key sections, in the order they appear in the script:

1. **DOM references** — `getElementById` lookups at the top of the IIFE. If you add an element, add its reference here.
2. **Storage keys** — `STORAGE_BEST`, `STORAGE_MUTE`, `STORAGE_DIFF`, `STORAGE_REDUCED`, all prefixed `skyfighter.`. Changing a key orphans players' saved data.
3. **Adaptive performance** — `trackPerf` samples frame times and steps a `quality` level (0 full, 1 medium, 2 low).
4. **Tuning tables** — `DIFFICULTIES` (spawn rates, enemy speed/fire chance, lives, bombs, boss HP multiplier), `KILLS_PER_STAGE`, `THEMES` (per-stage sky palette), `ENEMY_TYPES` (grunt, interceptor, tank), `BOSSES` (six bosses, cycled by stage).
5. **Audio** — Web Audio API, created lazily by `ensureAudio()`. Sounds come from `beep()` and `noiseBurst()`. Audio starts only after a user gesture.
6. **Input** — keyboard (`keydown`/`keyup` on `window`), mouse (steer and click-to-fire on the canvas), and touch (joystick in `#moveZone`, `#fireBtn`, `#bombBtn`).
7. **Game state and loop** — `resetGame()`, `update(dt)`, `draw()`, and `loop(now)`. `update` handles movement, firing, spawning, collisions, and power-ups. `draw` renders everything, including the HUD via DOM elements.
8. **Weapons and bosses** — `fireWeapon()` (vulcan/laser, levels 1–3), `useBomb()`, and boss attack patterns (`fireRadial`, `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`, `fireCross`, `fireHomingSeeded`, `fireDoubleRing`) driven by `updateBoss`.
9. **Flow control** — `loseLife()`, `endGame()`, `togglePause()`. Best score is written to `localStorage` in `endGame`.

### Controls (keep the on-screen hint in sync)

The hint text is `#controlsHint` in the markup. If you change a binding, update that text too.

- Move: Arrows or WASD; or mouse position on the canvas.
- Fire: Space, or hold the mouse button (fire button on mobile).
- Bomb: X or Shift (or the BOMB button).
- Pause: P or Escape.

## Conventions

- **Single file.** Keep all game code, styles, and markup in `index.html` unless the user asks for a split. Do not add build tooling.
- **Vanilla JS only.** No frameworks or external CDN dependencies. The page currently loads nothing external.
- **Tuning lives in the tables.** Prefer changing values in `DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, and `THEMES` over hard-coding numbers in logic.
- **Frame-rate independence.** Movement and timers use the `dt` passed to `update`. Keep new motion scaled by `dt`.
- **Accessibility.** Respect `reducedMotion` for shakes, flashes, and pulses. Announce important state changes through `announce()` (which feeds `#srLive`). Buttons need `title` or visible text.
- **Responsive layout.** The page must work on phones: keep touch targets usable, keep `touch-action: none` on the game surface, and respect `env(safe-area-inset-*)`.
- **Persistence.** Only `localStorage` is used, and every access should tolerate it being unavailable or throwing. Keep the `skyfighter.` key prefix.
- **Comments.** The existing code uses short section banners such as `// ---------- Section ----------`. Follow that style for new sections.

## Git workflow

- Develop on the branch specified by the task. Do not push to `main` unless explicitly asked.
- Write commit messages that say what changed and why, in the imperative mood.
- Do not create pull requests unless the user explicitly asks for one.
