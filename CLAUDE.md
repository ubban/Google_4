# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Google_4 is a single-file browser game: **Sky Fighter**, a Raiden-style vertical shoot-'em-up.

- `index.html` — the entire game: inline CSS, inline JavaScript (one IIFE), and the canvas-based renderer. There is no build step, no bundler, no dependencies, and no server component.
- `README.md` — a one-line title. Keep it accurate if the project description changes.

The git history has two commits: an initial commit and the Sky Fighter addition. Work happens on feature branches (see the session's branch instructions); do not push to `main` unless asked.

## Running and testing

- There is no build, lint, or test tooling. To run the game, open `index.html` directly in a browser, or serve the directory with any static server (for example `python3 -m http.server`).
- There are no automated tests. Verify gameplay changes by loading the page in a browser and playing through the affected path. Check the browser console for errors on load.
- Because there is no tooling, keep JavaScript valid in a modern browser without transpilation. Do not add dependencies or a build step without a clear reason.

## Code layout inside `index.html`

The script is one IIFE. Major sections, roughly in file order:

- **DOM references** — `getElementById` lookups near the top of the IIFE. Element ids are referenced from both the markup (`<div id="hud">`, `<div id="overlay">`, etc.) and the script; renaming an id requires updating both.
- **Storage keys** — `STORAGE_*` constants (`skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion`). Persisted through `localStorage`; reads and writes are not wrapped in try/catch today, so keep that in mind when touching them.
- **Tuning data** — `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES` (per-stage sky palettes), `ENEMY_TYPES` (grunt, interceptor, tank), and `BOSSES` (six boss designs, cycled by stage). Balance changes usually belong here rather than in the update logic.
- **Audio** — a lazily created `AudioContext` (`ensureAudio`), with `beep` and `noiseBurst` primitives and a `sfx` table of named sounds. Audio is created only after a user gesture.
- **Input** — keyboard (`keys` map), touch joystick and buttons, and mouse aiming on the canvas. Mouse, touch, and keyboard all feed the same `player` movement and firing state.
- **Rendering** — `drawPlane`, `drawBossShip`, and the `draw()` pass. Drawing is canvas 2D with gradients; the canvas is scaled to fit the viewport via `resize()` (base size 480×720).
- **Simulation** — `update(dt)` advances enemies, bullets, power-ups, bosses, and collisions. `spawnEnemy`, `spawnBoss`, and `maybeSpawnPowerUp` create entities.
- **Game loop** — `loop(now)` clamps `dt` to 50 ms and only updates and draws while the game is running and not paused.

## Gameplay facts to keep consistent

- Movement: arrow keys or WASD; firing: Space, or the on-screen FIRE button; bomb: X or Shift, or the BOMB button; pause: P or Escape.
- Difficulty (`easy` / `normal` / `hard`) sets lives, starting bombs, enemy fire rate, speed, spawn rate, and boss HP multiplier.
- A stage ends after `KILLS_PER_STAGE` (18) kills, then a boss spawns about 1.2 seconds later. Stage number selects the boss design and sky theme.
- Power-ups: life, bomb, and weapon (toggles between VULCAN and LASER). Weapon level is tracked on `player.level`.

## Conventions

- Match the surrounding style: two-space indentation, single quotes in JavaScript, and short comments only where the logic is not obvious.
- Keep the game self-contained in `index.html`. Do not split out files, add a framework, or pull in external scripts unless the user asks.
- Keep the page usable on phones: touch controls, `touch-action: none`, and the existing viewport meta tag should stay intact.
- Respect the reduced-motion setting: screen shake, flashes, and the heart pulse are gated by `reducedMotion`. New visual effects that move or flash should honor the same gate.
- Accessibility: announcements go through `announce()` and the `#srLive` live region. Use it for important state changes instead of adding new live regions.

## Git

- Develop on the branch named in the session instructions. Commit with clear messages that describe the gameplay or code change.
- Only push when the user asks, and never open a pull request unless the user explicitly requests one.
