# CLAUDE.md

Guidance for Claude Code when working in this repository.

## What this repo is

`Google_4` is a single-page HTML5 canvas game, **Sky Fighter** — a Raiden-style vertical shoot-'em-up. The entire game (markup, CSS, and JavaScript) lives in one file:

- `index.html` (~1,670 lines): `<style>` block at the top, body markup, then one `<script>` block (starts ~line 422) that contains all game logic.
- `README.md`: a one-line placeholder.

There is no build step, package manager, bundler, test suite, linter, or CI configuration. Do not add one unless asked.

## Running it

Open `index.html` directly in a browser, or serve the directory with any static server (e.g. `python3 -m http.server`). No install is needed.

## Code layout inside `index.html`

The script is a single IIFE-style block of `const`/`let` state and functions. Major sections, in source order:

- **DOM references** (~line 426): all `getElementById` lookups.
- **Storage keys** (~line 463): `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion`. Any new persisted setting should follow the `skyfighter.*` prefix and be read/written with `localStorage` guarded for failures.
- **Tuning tables** (~line 485–516): `DIFFICULTIES` (spawn rate, enemy speed, lives, bombs, boss HP multiplier), `KILLS_PER_STAGE` (18), `THEMES` (per-stage sky palettes, cycled by stage), `ENEMY_TYPES` (with `minStage` gating), `BOSSES` (cycled by stage). Prefer editing these tables over hard-coding numbers in logic.
- **Audio**: lazily created `AudioContext` (`ensureAudio`), with `beep`/`noiseBurst` primitives and a `sfx` map. Audio is only started after a user gesture.
- **HUD and feedback**: hearts/bombs rendering, hit flash, combo popups, stage banner, screen shake, vignette for low health.
- **Input** (~line 766): keyboard (`keys` map keyed by `e.code`), touch joystick and fire/bomb buttons, and mouse steering/click-to-fire on the canvas.
- **Game state** (~line 864): module-level `let` variables (`player`, `bullets`, `enemies`, `boss`, `score`, `lives`, `stage`, etc.) reset by `resetGame()`.
- **Drawing**: `drawPlane`, `drawBossShip`, and related canvas routines.
- **Spawning and update loop**: `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`, and the per-frame update/render loop driven by `requestAnimationFrame`.
- **Responsive scaling**: `resize()` scales a fixed `480×720` logical canvas (`BASE_W`/`BASE_H`) to the viewport, capped at 1.6×.
- **Adaptive quality**: `trackPerf` samples frame times and lowers `quality` (0 full, 1 medium, 2 low) on slow devices.

## Controls (keep the on-screen hint in sync)

The hint text lives in `#controlsHint` (~line 417). If you change a binding, update that string too.

- Mouse: move to steer, click to fire.
- Keyboard: Arrows/WASD move, Space fire, X or Shift bomb, P or Esc pause.
- Touch: drag left side to fly, tap FIRE, tap BOMB.

## Game rules worth knowing

- Lives and bombs depend on difficulty (`easy`, `normal`, `hard`).
- Each stage ends after `KILLS_PER_STAGE` kills, then a boss fight starts; boss HP scales with stage number and difficulty.
- Weapons: Vulcan and Laser. Weapon power-ups raise the level from 1 to 3; the next one swaps to the other weapon and resets the level to 1 (see the `type === 'weapon'` branch in the pickup handler).
- A kill-chain multiplier raises score; it resets when the chain timer runs out.
- `prefers-reduced-motion` is respected at startup and can be toggled; reduced motion disables shake/flash/pulse effects.

## Conventions

- Keep everything in `index.html` unless the user asks for a split. Do not introduce external CDN scripts or fonts without a reason; the game is intended to run offline from a single file.
- Keep changes small and in the existing style: 2-space indent, `const` by default, single quotes in JS, and no new dependencies.
- Preserve mobile behavior: touch controls must keep working, and `touch-action: none` on the game surface must stay.
- Preserve accessibility hooks: announcements go through `announce()` into the `srLive` region, and reduced-motion must keep suppressing the effects it currently suppresses.
- Do not change the `localStorage` key names; existing players' best score and settings depend on them.

## Verifying changes

No automated tests exist. For gameplay or rendering changes, load `index.html` in a browser and play through at least one stage transition and a boss fight, on both keyboard and a touch-sized viewport. Check the browser console for errors.

## Git

- Develop on the branch assigned for the task (currently `claude/gifted-tesla-fbbayl`). Do not push to `main` directly.
- Do not open a pull request unless explicitly asked.
