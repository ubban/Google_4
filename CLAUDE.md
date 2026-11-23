# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Google_4 is a small, dependency-free web project. Its only real content is
**Sky Fighter**, a Raiden-style vertical shoot-'em-up implemented as a single
HTML5 canvas page.

- `index.html` — the entire game: markup, CSS (inline `<style>`), and
  JavaScript (one inline `<script>` wrapped in an IIFE). Roughly 1,700 lines.
- `README.md` — minimal project readme (currently a placeholder).

There is no build step, no package manager, no bundler, and no test suite.

## Running the game

Open `index.html` directly in a browser, or serve the directory with any
static server (for example `python3 -m http.server` and visit
`http://localhost:8000`). No install is required.

## Code layout inside `index.html`

The script is organized roughly top to bottom as:

1. **Constants and config** — storage keys (`STORAGE_*`), `DIFFICULTIES`,
   `KILLS_PER_STAGE`, `THEMES` (per-stage sky palettes), `ENEMY_TYPES`,
   `BOSSES`, `BASE_W`/`BASE_H` (logical canvas size, 480×720).
2. **Persistence and settings** — best score, mute, difficulty, and reduced
   motion are read from and written to `localStorage`.
3. **Audio** — a lazily created Web Audio context (`ensureAudio`) with
   `beep` and `noiseBurst` helpers. Audio must only start after a user gesture.
4. **UI helpers** — HUD rendering (hearts, bombs), flashes, combo pops, stage
   banners, screen shake.
5. **Input** — keyboard (`keys[e.code]`), pointer/touch, and a virtual
   joystick for mobile.
6. **Game state and logic** — `resetGame`, `update(dt)`, weapons
   (`fireWeapon`), power-ups, smart bombs (`useBomb`), kill chain (`addScore`,
   `breakChain`), enemy and boss spawning, and collisions (`rectsOverlap`).
7. **Bosses** — `BOSS_PATTERNS` are bullet-pattern functions (radial, spread,
   spiral, cross, homing, and so on) driven by `updateBoss`.
8. **Rendering** — `draw()` and helpers (`drawPlane`, `drawBossShip`,
   `drawClouds`). Parallax clouds and sky gradients follow the current theme.
9. **Main loop** — `loop(now)` drives `update` and `draw` via
   `requestAnimationFrame`, with a delta-time cap.

Keep new code in the same IIFE and follow the existing naming style
(`camelCase` functions, `UPPER_SNAKE_CASE` constants).

## Controls (for reference when testing)

- Move: arrow keys / WASD, or drag / virtual joystick on touch.
- Fire: Space (hold), or touch.
- Bomb: `X` or Shift.
- Pause: `P` or Escape.
- Mute and reduced motion are toggled from the in-game UI.

## Conventions

- Keep everything in `index.html` unless there is a clear reason to split
  files. Do not add a framework or build tooling without asking.
- Preserve mobile support: the layout uses `viewport-fit=cover`, safe-area
  insets, and `touch-action: none`. Check that changes still work at phone
  width.
- Keep the fixed logical resolution (`BASE_W` × `BASE_H`) when changing
  rendering or collision code; `resize()` scales the canvas to fit the window.
- Persisted keys use the `skyfighter.*` prefix. Wrap new `localStorage` access
  in a try/catch, since storage can be unavailable (private windows, blocked
  site data).
- Balance numbers live in `DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, and
  `BOSS_PATTERNS`. Prefer tuning them there over hard-coding values in logic.

## Verifying changes

There are no automated tests. For gameplay or rendering changes:

1. Load `index.html` in a browser and check the browser console for errors.
2. Play through at least one stage start to boss, and confirm pause, bomb,
   game over, and best-score persistence still work.
3. Check the layout at a narrow (phone) viewport as well as desktop.

## Git workflow

- Develop on the branch assigned for the session (currently
  `claude/gifted-tesla-iczwej`). Do not push to other branches without
  explicit permission.
- Do not open a pull request unless the user asks for one.
- Commit messages should be short and imperative, with a body describing the
  user-visible change when it is non-trivial.
