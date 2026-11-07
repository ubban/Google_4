# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo contains **Sky Fighter**, a Raiden-style vertical shoot-em-up built as a
single-file HTML5 canvas game.

- Everything lives in `index.html`: markup (`<body>`, ~line 361), CSS (`<style>`,
  ~line 10), and one IIFE-wrapped `<script>` (~line 422) with all game logic.
- No build step, no bundler, no package manager, no tests. Open the file in a
  browser (or serve the directory statically) to run it.
- `README.md` is a placeholder; treat `index.html` as the source of truth.

## Running and checking changes

- Run: open `index.html` directly, or serve the folder (e.g. `python3 -m http.server`)
  and visit it in a browser.
- There is no automated test suite or linter configured. Verify changes by playing
  the game in a browser, including the affected code path (e.g. boss fights, pause,
  mobile controls) and by checking the browser console for errors.
- Keep the file self-contained: do not add external scripts, stylesheets, or fonts
  unless the user asks for them.

## Code structure (inside the `<script>` block)

Sections appear roughly in this order:

1. **Persistence keys** — `STORAGE_BEST`, `STORAGE_MUTE`, `STORAGE_DIFF`,
   `STORAGE_REDUCED` (all `localStorage`, prefixed `skyfighter.`).
2. **Tuning tables** — `DIFFICULTIES` (easy/normal/hard: spawn rates, enemy speed,
   enemy fire chance, lives, bombs, boss HP multiplier), `KILLS_PER_STAGE`
   (currently 18), `THEMES` (per-stage sky palettes, cycled by stage), `ENEMY_TYPES`
   (grunt, interceptor, tank), `BOSSES` (six boss designs with HP multipliers).
3. **Audio** — `ensureAudio`, `beep`, `noiseBurst`, and an `sfx` object of named
   sound effects. Audio is created lazily on the first user gesture.
4. **HUD and feedback helpers** — hearts/bombs rendering, `announce` (screen-reader
   live region), combo pops, stage banners, screen shake, hit/bomb flashes.
5. **Input** — keyboard (`keys` map), mouse (canvas steering/firing), and touch
   joystick + fire/bomb buttons. Touch controls only appear on touch devices.
6. **Game state and logic** — `resetGame`, `spawnEnemy`, `spawnBoss`,
   `maybeSpawnPowerUp`, `update(dt)`, `applyPowerUp`, `useBomb`, `fireWeapon`,
   `loseLife`, `endGame`, `togglePause`.
7. **Boss patterns** — `BOSS_PATTERNS` and the `fire*` helpers (radial, aimed spread,
   columns, spiral, cross, homing, double ring) driven by `updateBoss`.
8. **Rendering** — `draw()` plus `drawPlane`, `drawBossShip`, `drawClouds`.
9. **Main loop** — `loop(now)` via `requestAnimationFrame`, with `dt` clamped and
   a perf-tracking hook (`trackPerf`).

Game-world coordinates use a fixed `BASE_W = 480` × `BASE_H = 720` logical
resolution; `resize()` scales the canvas to the window.

## Gameplay facts to keep consistent

- **Weapons:** Vulcan and Laser, each with 3 levels. A weapon power-up raises the
  level; at level 3 it switches weapon and resets to level 1.
- **Power-ups:** `life` (+1 life, capped), `bomb` (+1 bomb, capped), `weapon`.
- **Bombs:** cleared enemies on screen and damage bosses. Used via `X`, `Shift`, or
  the on-screen BOMB button.
- **Kill chain:** consecutive kills build a score multiplier. `breakChain` resets it
  when the player loses a life or when the chain timer runs out.
- **Stages:** after `KILLS_PER_STAGE` kills in a stage, a boss spawns
  (`spawnBoss`, which picks `BOSSES[(stage - 1) % BOSSES.length]`). Defeating the boss
  (`killBoss`) increments `stage`; the theme comes from `THEMES[(stage - 1) % THEMES.length]`.
- **Controls:** Arrows/WASD move, Space fires (mouse click also fires), X/Shift
  bomb, P/Esc pause.
- **Best score** persists in `localStorage`.

When changing a tuning value, update the table entry rather than hardcoding a
number in logic. Keep the three difficulty presets balanced relative to each other.

## Conventions

- Plain JavaScript (ES2015+ is fine; the file already uses arrow functions,
  `const`/`let`, template-free DOM APIs). No frameworks, no transpilation.
- Keep everything inside the existing IIFE; don't add new globals.
- Follow the existing style: 2-space indentation, single quotes in JS, section
  banner comments like `// ---------- Input ----------`.
- Prefer extending existing tables (`ENEMY_TYPES`, `BOSSES`, `BOSS_PATTERNS`,
  `THEMES`, `DIFFICULTIES`) over adding parallel special-case code.
- Any DOM element referenced from JS must exist in the markup and should be looked up
  once near the top of the script.
- Respect `reducedMotion`: new screen effects (shake, flashes, particles) should be
  gated or reduced when it is on.
- Keep audio and visual feedback paired with gameplay events, and route user-facing
  status text through `announce` for accessibility.
- Preserve mobile support: any new control needs a touch path as well as keyboard
  and mouse.

## Git workflow

- Develop on the branch specified by the task (currently `claude/gifted-tesla-48fbi5`).
  Do not push to other branches without explicit permission.
- Commit with clear, descriptive messages. Push with `git push -u origin <branch>`.
- Do not open a pull request unless the user explicitly asks for one.
