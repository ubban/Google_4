# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Google_4 is a small static web project. Its only real content is **Sky Fighter**, a Raiden-style vertical shoot-em-up written as a single-page HTML5 canvas game with no build step, no package manager, and no external dependencies.

## Repository layout

- `index.html` — the entire game: markup, CSS (`<style>` block, roughly lines 10–361), canvas element, and one inline `<script>` IIFE (roughly lines 422–1668) containing all game logic.
- `README.md` — one-line placeholder; keep it accurate if the project description changes.
- `CLAUDE.md` — this file.

There is no `package.json`, test suite, linter, or CI configuration in the repo.

## Running and verifying

- Open `index.html` directly in a browser, or serve the directory with any static server (e.g. `python3 -m http.server`).
- There are no automated tests. Verify changes by loading the page and playing: start a run, fire, use a bomb, take a hit, pause, change difficulty, and check the game-over screen.
- Test on a narrow viewport as well as desktop, because touch controls render on mobile.

## Architecture of the game script

All game state lives in closures inside the IIFE. Key pieces, in file order:

- **DOM references** — cached at the top of the script; HUD and overlay elements are looked up by ID, so renaming an ID in the markup breaks the script silently.
- **Persistence** — `localStorage` keys prefixed `skyfighter.` (`bestScore`, `muted`, `difficulty`, `reducedMotion`). Every access should tolerate storage being unavailable.
- **Adaptive quality** — `trackPerf` samples frame times and steps a `quality` level (0–2) up or down to reduce work on slow devices.
- **Config tables** — `DIFFICULTIES` (spawn rate, enemy speed and fire chance, lives, bombs, boss HP multiplier), `THEMES` (per-stage sky/cloud/sun palettes), `ENEMY_TYPES` (grunt, interceptor, tank, with minimum stage), and `BOSSES` (six bosses, cycled per stage). Balance changes should usually go here rather than in logic.
- **Audio** — Web Audio synthesis (`beep`, `noiseBurst`); audio is created lazily by `ensureAudio()` on the first user gesture, which browsers require.
- **Input** — keyboard (`keys[e.code]`), mouse/touch fire button, and a virtual joystick on touch devices. Keys: arrows or WASD move, Space fires, X or Shift bombs, P or Escape pauses.
- **Simulation** — `update(dt)` advances entities; `resetGame()` restores the initial state for a new run; `loseLife()`, `endGame()`, `killBoss()`, and `togglePause()` handle state transitions.
- **Weapons and power-ups** — `fireWeapon()` and `applyPowerUp()` implement the Vulcan/Laser weapon with three power levels, plus bombs and extra lives.
- **Bosses** — `BOSS_PATTERNS` is an array of bullet-pattern functions (radial, aimed spread, columns, spiral, cross, homing, double ring) that `updateBoss()` cycles through.
- **Rendering** — `draw()` paints everything each frame; `drawPlane`, `drawBossShip`, and `drawClouds` are the sprite helpers.
- **Loop** — `loop(now)` clamps `dt` to 50 ms, runs `update` and `draw` only while running and not paused, and reschedules via `requestAnimationFrame`.

## Conventions

- Keep everything in `index.html` unless the project grows enough to justify splitting files. If you split, update this section.
- Match the existing style: two-space indentation, single quotes in JS, `const` by default, section headers written as `// ---------- Name ----------`, and short comments that explain intent rather than restating code.
- Keep the code in the IIFE and avoid new globals. Expose nothing on `window` unless there is a clear need.
- Keep the game dependency-free. Do not add frameworks or CDN scripts without an explicit request.
- Preserve the accessibility hooks: the `srLive` live region for announcements, the `reduced-motion` body class, and the `prefers-reduced-motion` default.
- Keep the mobile layout working: the viewport meta disables zoom, and `touchControls` must stay usable with touch-action rules intact.

## Git workflow

- Develop on the branch specified by the task (currently `claude/gifted-tesla-fe31wk`). Do not push to `main` directly.
- Write descriptive commit messages in the imperative mood, with a short summary line and a body for non-trivial changes.
- Do not open a pull request unless explicitly asked.
