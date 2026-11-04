# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a small, dependency-free browser game, **Sky Fighter**: a Raiden-style vertical shoot-'em-up. The whole game lives in one file:

- `index.html` (~1,670 lines) — markup, CSS (`<style>` block, ~lines 10–362), and a single inline `<script>` (starts ~line 422) containing all game logic, rendering, audio, and input.
- `README.md` — minimal placeholder; no setup instructions.

There is no `package.json`, build step, bundler, linter, or test suite. Nothing needs to be installed.

## Running the game

Open `index.html` directly in a browser (double-click, or serve the folder with any static server, e.g. `python3 -m http.server`). No build is required.

## Verifying changes

There are no automated tests. After editing, verify by:

1. Loading `index.html` in a browser with no console errors.
2. Playing through the affected code path (start, fire, bomb, pause, weapon switch, stage transition, boss, game over).
3. Checking both desktop (keyboard/mouse) and touch layouts if the change touches input or the HUD.

Keep the game responsive at phone widths; the canvas scales to the viewport (see `resize()`).

## Code structure (inside `index.html`)

The inline script is organized roughly in this order:

- **DOM references** — `getElementById` lookups at the top of the script.
- **Storage keys** — `STORAGE_BEST`, `STORAGE_MUTE`, `STORAGE_DIFF`, `STORAGE_REDUCED` (`skyfighter.*` in `localStorage`).
- **Config tables** — `DIFFICULTIES` (easy/normal/hard: spawn rates, enemy speed, shot chance, lives, bombs, boss HP multiplier), `KILLS_PER_STAGE` (18), `THEMES` (six stage palettes that cycle by stage), `ENEMY_TYPES` (grunt, interceptor, tank — with `minStage` gating), and `BOSSES` (six bosses that cycle by stage, each with an `hpMul`).
- **Audio** — lazily created `AudioContext` (`ensureAudio`), with `beep`, `noiseBurst`, and a named `sfx` table. Respect the mute flag.
- **Feedback helpers** — hearts/bombs HUD rendering, screen shake, flash effects, combo pop, stage banner, and `announce()` for screen readers (`srLive`).
- **Input** — keyboard (`keys` map on `keydown`/`keyup`), touch joystick + fire/bomb buttons, and mouse/pointer targeting on the canvas.
- **Game state** — module-level `let` variables (`player`, `bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`, `boss`, `score`, `lives`, `bombCount`, `stage`, `chain`/`multiplier`, etc.). `resetGame()` reinitializes them.
- **Simulation** — `update(dt)` (per-frame logic, collisions, spawning), `updateBoss(dt)`, `spawnEnemy()`, `spawnBoss()`, `maybeSpawnPowerUp()`.
- **Rendering** — `draw()`, `drawPlane()`, `drawBossShip()`, `drawClouds()`. Drawing is done with the 2D canvas context on a fixed `BASE_W × BASE_H` (480 × 720) logical size.
- **Main loop** — `loop(now)` drives `update` and `draw` via `requestAnimationFrame`. `trackPerf()` adapts a `quality` level (full/medium/low) if frame times degrade.

## Key conventions

- **Single file, no modules.** Keep new code inside the existing `<script>` and `<style>` blocks unless the user explicitly asks to split the project. Do not introduce a build step or external dependencies without asking.
- **Config over magic numbers.** Tune difficulty, enemy stats, boss stats, and stage palettes through the existing config objects rather than hard-coding values in logic.
- **Persist only through the `skyfighter.*` keys.** Wrap `localStorage` access so the game still works if storage is unavailable (private browsing, blocked storage).
- **Respect user preferences.** Honor the mute setting and reduced-motion (`prefers-reduced-motion` plus the in-game toggle) when adding effects, shake, or flashes.
- **Accessibility.** Announce important state changes through `announce()`. Keep buttons labeled and keyboard-reachable.
- **Pause handling.** Gameplay logic should respect `paused` and `gameOver` flags.
- **Frame-rate independence.** Movement and timers use the `dt` passed to `update`; keep new timers consistent with that.

## Controls (for reference when testing)

- Move: arrow keys / WASD, mouse (steer by moving, click to fire), or touch (drag left side to fly)
- Fire: Space (held), mouse click, or FIRE button on touch
- Bomb: X or Shift, or BOMB button on touch
- Pause: P or Escape
- Weapon pickups (`weapon` type) first raise the current weapon level 1→3; the next pickup past level 3 switches between `vulcan` and `laser` and resets the level to 1.

## Git workflow

- Develop on the branch assigned for the session (currently `claude/gifted-tesla-q9j40a`). Do not push to other branches without explicit permission.
- Use clear, descriptive commit messages.
- Do not open pull requests unless explicitly asked.

## Things to avoid

- Do not add a package manager, framework, or bundler without an explicit request.
- Do not rename the `skyfighter.*` storage keys; doing so resets players' saved best scores and preferences.
- Do not remove the `prefers-reduced-motion` handling or the mute/pause controls.
