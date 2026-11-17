# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a small, dependency-free browser game, **Sky Fighter**: a Raiden-style vertical shoot-'em-up with weapon switching, smart bombs, stage bosses, and difficulty levels.

The entire game lives in one file, `index.html` (~1,670 lines). It contains the inline `<style>`, the markup, and one inline `<script>`. There is no build step, no package manager, no test suite, and no external assets. `README.md` is a one-line title.

## Running it

- Open `index.html` directly in a browser, or serve the directory with any static server (for example `python3 -m http.server` and visit `http://localhost:8000/index.html`).
- Changes are visible on reload. There is nothing to compile.

## Repository layout

| Path | Purpose |
| --- | --- |
| `index.html` | Entire game: styles, HUD/overlay markup, and game logic. |
| `README.md` | Project title. |
| `CLAUDE.md` | This file. |

## Architecture of `index.html`

Structure within the inline script (approximate line numbers, which will drift):

- **Config constants** (~L463–515): `STORAGE_*` keys, `DIFFICULTIES` (easy/normal/hard), `KILLS_PER_STAGE`, `THEMES` (sky palettes cycled per stage), `ENEMY_TYPES` (grunt, interceptor, tank), and `BOSSES` (six bosses, each with HP multiplier and colors).
- **Persistence**: `localStorage` keys `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, and `skyfighter.reducedMotion`. Reads are not wrapped in try/catch today; keep this in mind if you touch them.
- **Audio** (~L543–680): a lazily created `AudioContext` (`ensureAudio`), synthesized sound via `beep` and `noiseBurst`. Nothing is loaded from disk.
- **Layout and input** (~L753–865): `resize()` scales a fixed `BASE_W × BASE_H` (480×720) canvas to fit the viewport. Input comes from the keyboard, a touch joystick plus FIRE/BOMB buttons, and mouse. Touch controls are shown only on touch devices.
- **Game state and loop** (~L865–1150): `resetGame()`, `update(dt)`, `fireWeapon()`, `useBomb()`, scoring and combo chains (`addScore`, `breakChain`, `multiplier`).
- **Bosses** (~L1304–1455): `BOSS_PATTERNS` are bullet-pattern functions (radial, spread, columns, spiral, cross, homing, double ring) driven by `updateBoss`. `killBoss` and `loseLife` handle progression, and `endGame` shows the overlay.
- **Rendering** (~L913–1640): canvas drawing helpers (`drawPlane`, `drawBossShip`, `drawClouds`) and `draw()`. `trackPerf` adjusts a `quality` level (0 full, 1 medium, 2 low) when frames are slow.
- **Main loop**: `loop(now)` uses `requestAnimationFrame` with a delta-time `dt`.

### Controls (for reference when editing input code)

- Move: arrow keys or WASD. Fire: Space, or hold the FIRE button or mouse button.
- Bomb: X or Shift. Pause: P or Escape.
- Touch: left-side joystick to move, FIRE and BOMB buttons on the right.

## Conventions

- **Single file, no dependencies.** Keep it that way unless the user asks otherwise. Do not add a bundler, framework, or npm packages without a clear request.
- **Keep the existing style**: 2-space indentation, `const`/`let`, plain functions, and comments in `// ---------- Section ----------` form.
- **Keep the fixed logical resolution.** Game coordinates are in `BASE_W × BASE_H`; scale only through `resize()`.
- **Respect the reduced-motion setting.** Screen shake, flashes, and pulses should check `reducedMotion` as the existing code does.
- **Accessibility hooks**: announcements go through `announce()` into the `#srLive` live region. Keep buttons labeled.
- **Mobile**: the page disables scroll and pinch zoom and uses `touch-action: none`. Test layout changes at phone width.

## Verifying changes

There are no automated tests or linters. Verify by:

1. Loading `index.html` in a browser and checking the console for errors.
2. Playing through start → stage 1 → boss, and checking HUD updates, pause/resume, bombs, and game over.
3. For layout or input changes, testing at both desktop and phone viewport sizes, and with touch emulation if possible.

If you add a test harness or tooling, document the command here.

## Git

- Work on the branch you were assigned (currently `claude/gifted-tesla-vsv0aj`). Do not push to other branches without permission.
- Keep commits focused, with short imperative messages.
