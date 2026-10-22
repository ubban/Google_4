# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo contains **Sky Fighter**, a Raiden-style vertical shoot-em-up built as a
single-file HTML5 canvas game. There is no build step, no package manager, and no
test suite.

| File        | Purpose                                                        |
|-------------|----------------------------------------------------------------|
| `index.html`| The entire game: markup (~lines 1–420), CSS (~lines 10–362), and JavaScript (~lines 422–1669) |
| `README.md` | Placeholder; no setup instructions                             |

## Running the game

No install or build is needed. Open `index.html` directly in a browser, or serve the
directory with any static server (e.g. `python3 -m http.server`) and visit it.

There are no automated tests or linters configured. Verify changes by playing the game
in a browser, ideally at both desktop and phone-sized viewports, since the touch
controls are only shown on touch devices.

## Code architecture

All game code lives in one IIFE inside the inline `<script>` tag. Key sections, in
order of appearance:

- **DOM refs and constants**: element lookups and `STORAGE_*` localStorage keys
  (`skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`,
  `skyfighter.reducedMotion`).
- **Tuning tables**: `DIFFICULTIES` (spawn rate, enemy speed, shot chance, lives,
  bombs, boss HP multiplier), `THEMES` (per-stage sky palette, cycled by stage),
  `ENEMY_TYPES` (grunt / interceptor / tank), `BOSSES` (six boss variants),
  `KILLS_PER_STAGE` (18).
- **Audio**: a lazily created `AudioContext` (`ensureAudio`) with `beep` and
  `noiseBurst` helpers. Mute state persists in localStorage.
- **Input**: keyboard handlers on `window`, touch joystick and fire/bomb buttons, and
  mouse/pointer tracking on the canvas. `canvasPointFromEvent` maps client coordinates
  into the fixed `BASE_W × BASE_H` (480 × 720) logical space; `resize` scales the
  canvas to fit the viewport.
- **Game state**: `resetGame` initializes the run. `update(dt)` advances the simulation
  (player, bullets, enemies, power-ups, collisions, stages). `draw()` renders each frame.
- **Weapons and bosses**: `fireWeapon` (Vulcan/Laser, 3 power levels), `useBomb`
  (screen-clearing, damages bosses), `BOSS_PATTERNS` (bullet-pattern functions such as
  radial, spiral, cross, and homing), `updateBoss` / `killBoss`.
- **Lifecycle**: `loseLife`, `endGame`, `togglePause`, and the `loop(now)` driver.
  `loop` clamps `dt` to 50 ms and runs `update` and `draw` only while the game is
  running and not paused.
- **Adaptive quality**: `trackPerf` averages the last 45 frame times and steps a
  `quality` level (0 full, 1 medium, 2 low) to keep frame rate up on slow devices.

### Controls

- Move: arrow keys or WASD; touch joystick on mobile; mouse pointer over the canvas.
- Fire: Space; hold the mouse button on the canvas or the fire button on touch.
- Bomb: X or Shift (touch: bomb button).
- Pause: P or Escape.

## Conventions

- Keep the game self-contained in `index.html`. Do not add build tooling, frameworks,
  or external dependencies unless the user asks for them.
- Follow the existing style: `const`/`let`, arrow functions for callbacks, two-space
  indentation, and `// ----------` section banners for grouping.
- Put balance changes in the existing tuning tables (`DIFFICULTIES`, `ENEMY_TYPES`,
  `BOSSES`, `THEMES`) instead of hard-coding numbers in logic.
- Use the `STORAGE_*` constants for any new persisted setting. Existing reads and
  writes are not guarded, so wrap new access in try/catch if it may run where storage
  is blocked.
- Respect the reduced-motion setting: `screenShake` dampens its magnitude and duration,
  and the hit and bomb flashes return early. New visual effects should check
  `reducedMotion` too.
- Announce important state changes through `announce()` so screen-reader users get
  the update via the `srLive` region.

## Git

- Development branch for this work: `claude/gifted-tesla-izz1ff`.
- Write clear, descriptive commit messages that explain why a change was made.
