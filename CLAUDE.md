# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Google_4 is a small static web project. Its main content is **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs entirely in the browser as a single HTML file.

Files:

- `index.html` — the whole game: inline CSS (`<style>`, roughly lines 10–361), HTML/UI markup (roughly lines 362–421), and one inline `<script>` (roughly lines 422–1668) wrapped in an IIFE with `'use strict'`.
- `README.md` — a one-line placeholder title. Nothing else.

There is no build step, no package manager, no test suite, no linter config, and no dependencies. Do not introduce any of these without being asked.

## Running and verifying

- Open `index.html` directly in a browser, or serve the directory with any static server (for example `python3 -m http.server`).
- There are no automated tests. Verify changes by playing the game in a browser: start a run, check movement, firing, bombs, power-ups, boss fights, pause, mute, reduced-motion toggle, and the game-over screen.
- Check both desktop (keyboard) and touch layouts when touching input or HUD code. Touch controls are shown only when `'ontouchstart' in window` or `navigator.maxTouchPoints > 0`.

## Code structure (inside `index.html`)

Everything lives in the single IIFE. Key sections, in file order:

- **DOM references** — `document.getElementById` lookups at the top of the script.
- **Storage keys** — `STORAGE_BEST`, `STORAGE_MUTE`, `STORAGE_DIFF`, `STORAGE_REDUCED`, all prefixed `skyfighter.` in `localStorage`.
- **Adaptive performance** — `trackPerf` lowers a `quality` level (0 full, 1 medium, 2 low) when frame times are poor.
- **Config tables** — `DIFFICULTIES` (easy/normal/hard tuning), `THEMES` (six sky palettes, one per stage cycle), `ENEMY_TYPES` (grunt, interceptor, tank), `BOSSES` (six boss definitions), `KILLS_PER_STAGE` (18).
- **Audio** — a Web Audio `sfx` object built from `beep` and `noiseBurst`. Audio is created lazily by `ensureAudio()`, which browsers require to be triggered by a user gesture.
- **Input** — `keys` map from `KeyboardEvent.code`, touch joystick, fire and bomb buttons, and mouse handlers for the canvas.
- **Game state and logic** — `resetGame`, `update(dt)`, `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`, `applyPowerUp`, `fireWeapon`, `useBomb`, `updateBoss`, `loseLife`, `endGame`, `togglePause`.
- **Boss attack patterns** — `BOSS_PATTERNS` and the `fire*` helpers (radial, aimed spread, columns, spiral, cross, homing, double ring).
- **Rendering** — `drawPlane`, `drawBossShip`, `drawClouds`, `draw()`. Drawing uses the 2D canvas context directly.
- **Main loop** — `loop(now)` runs on `requestAnimationFrame`. `dt` is clamped to 0.05 s.

The logical play field is `BASE_W = 480` by `BASE_H = 720`; `resize()` scales the canvas to fit the viewport.

## Controls (for reference when changing input)

- Move: arrow keys or WASD.
- Fire: Space (hold), or the FIRE button / mouse button on the canvas.
- Bomb: X or Shift, or the BOMB button.
- Pause: P or Escape.
- Power-up types: `life`, `bomb`, and `weapon` (toggles between `vulcan` and `laser`).

## Conventions

- Keep the project as one self-contained `index.html` unless the user asks to split it. Do not add external CDN scripts or fonts without asking.
- Match the existing style: 2-space indentation, single quotes in JS, `const` by default, section headers written as `// ---------- Name ----------`.
- Put new tuning values in the existing config tables (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `THEMES`) rather than hard-coding magic numbers in logic.
- Persist user settings only through the existing `skyfighter.*` `localStorage` keys, and wrap any new storage use so the game still works when storage is unavailable.
- Keep the UI accessible: announce important state changes through `announce()` (the `srLive` region), and respect the reduced-motion setting for screen shake, flash, and pulse effects.

## Git

- Development branch for this work: `claude/gifted-tesla-atieyh`. Do not push to other branches without explicit permission.
- Do not create a pull request unless the user asks for one.
- Commit only when asked.
