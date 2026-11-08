# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Google_4 is a small, dependency-free browser game: **Sky Fighter**, a Raiden-style
vertical shoot-'em-up. The entire game ships as one static file:

- `index.html` (~1.7k lines) — inline `<style>`, the HTML/HUD markup, and one inline
  `<script>` IIFE containing all game logic, rendering, audio, and input.
- `README.md` — minimal project title only.

There is no build step, package manager, test suite, or linter configured.

## Running locally

Open `index.html` directly in a browser, or serve the directory with any static server:

```sh
python3 -m http.server 8000   # then visit http://localhost:8000
```

Verify changes by playing the game in a browser. Check at least: starting a run,
firing, a bomb, a boss fight, pausing, game over, and the high-score persistence.
Also check the mobile layout (touch controls appear automatically on touch devices).

## Code layout (inside `index.html`)

The script is wrapped in an IIFE with `'use strict'`. Major sections, in source order:

| Area | Where to look |
| --- | --- |
| DOM references | Top of the script: `document.getElementById(...)` constants |
| Storage keys | `STORAGE_*` constants (`skyfighter.*` in `localStorage`) |
| Adaptive quality | `trackPerf` / `quality` (0 full, 1 medium, 2 low) |
| Tuning data | `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES`, `ENEMY_TYPES`, `BOSSES` |
| Audio | `ensureAudio`, `beep`, `noiseBurst`, `EXPLODE_SFX` (Web Audio, no asset files) |
| HUD helpers | `renderHearts`, `renderBombs`, `showComboPop`, `showStageBanner`, `announce` |
| Layout / scaling | `BASE_W = 480`, `BASE_H = 720`, `resize()` |
| Input | Keyboard handlers, joystick and touch handlers, `canvasPointFromEvent` |
| Game state | `resetGame()`; state vars `player`, `bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`, `boss` |
| Spawning | `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp` |
| Update step | `update(dt)` — movement, collisions, stage progression, power-up pickup |
| Weapons & bosses | `fireWeapon`, `applyPowerUp`, `BOSS_PATTERNS` and the `fire*` pattern helpers, `updateBoss`, `killBoss` |
| Lifecycle | `loseLife`, `endGame`, `togglePause` |
| Rendering | `draw()`, `drawPlane`, `drawBossShip`, `drawClouds` |
| Main loop | `loop(now)` — `dt` is clamped to 50 ms; `update` then `draw` only while running and not paused |

Keyboard controls: arrows/WASD-style movement via `keys[e.code]`, `Space` fires,
`X` or `Shift` drops a bomb, `P` or `Esc` pauses.

## Conventions

- **Keep it a single file.** Do not split the game into modules, add a bundler, or
  introduce dependencies unless the user explicitly asks. Everything stays inline.
- **No external assets.** Graphics are drawn with the Canvas 2D API and sound is
  synthesized with Web Audio. Do not add image or audio files without a request.
- **Match the existing style:** `const`/`let`, arrow functions for callbacks,
  two-space indentation, and short descriptive helper names. Use the section
  divider comments (`// ---------- Name ----------`) when adding a new section.
- **Tune through data tables.** Balance changes (enemy stats, difficulty, boss HP,
  stage length) belong in the constant tables near the top of the script, not
  hard-coded in logic.
- **Respect the time step.** Movement and timers use `dt` (seconds). Do not assume a
  fixed frame rate.
- **Accessibility and preferences.** Honor `reducedMotion` (also set from
  `prefers-reduced-motion`) when adding screen shake, flashes, or particle effects.
  Announce important state changes through `announce()` for screen readers.
- **Persistence is local only.** Best score, mute, difficulty, and reduced-motion
  settings use `localStorage` with the `skyfighter.*` keys. Wrap new storage access
  so it still works if storage is unavailable.
- **Mobile first-class.** Keep the layout working at small widths and preserve
  `touch-action`/`preventDefault` behavior on the on-screen controls.

## Making changes

1. Read the surrounding section before editing; most logic is tightly coupled through
   shared module-level state.
2. Make the smallest change that does the job. Avoid refactors or renames unrelated to
   the task.
3. Test in a browser, including the keyboard and touch paths you touched.
4. Commit with a clear, descriptive message and push to the branch you were given.

## Out of scope unless requested

- Adding a framework, build tooling, test harness, or CI workflow.
- Changing the game's name, theme, or the 480×720 base resolution.
