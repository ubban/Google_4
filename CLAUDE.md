# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a small static web project. Its only game is **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs in the browser on an HTML5 canvas.

- Everything lives in one file, `index.html`: markup (`<body>`, ~line 362 onward), CSS (`<style>`, ~line 10), and one IIFE holding all game code (`<script>`, ~line 422 onward).
- `README.md` is a placeholder with only the project title.
- There is no build step, package manager, bundler, or test suite. Do not add one unless asked.
- No external libraries or CDN assets are used. Keep it that way.

## Running and verifying

- Open `index.html` directly in a browser, or serve the directory with any static server (for example `python3 -m http.server`).
- There are no automated tests. To verify a change, play it in a browser: start a game, check the affected system, and confirm there are no console errors.
- For headless checks, Playwright with the pre-installed Chromium works. See the environment notes for the browser path.

## Code layout inside `index.html`

The game script is a single IIFE. Major sections, in source order:

1. **DOM references**: `const xxxEl = document.getElementById(...)`.
2. **Storage keys**: `STORAGE_BEST`, `STORAGE_MUTE`, `STORAGE_DIFF`, `STORAGE_REDUCED` (all prefixed `skyfighter.`).
3. **Accessibility and perf**: `announce()` writes to the `#srLive` live region; `trackPerf()` tracks frame times.
4. **Config tables**:
   - `DIFFICULTIES` (`easy` / `normal` / `hard`): spawn rates, enemy speed, shoot chance, starting lives and bombs, boss HP multiplier.
   - `KILLS_PER_STAGE` (18).
   - `THEMES`: per-stage sky gradient, cloud and sun colors, and names. Cycled by stage.
   - `ENEMY_TYPES`: `grunt`, `interceptor` (weaves), and `tank` (4 HP, appears from stage 2).
   - `BOSSES`: six bosses (VANGUARD through TITAN), each with colors and an `hpMul`.
5. **Persistent state**: `bestScore`, `muted`, `difficulty`, `reducedMotion`, read from `localStorage` with fallbacks.
6. **Audio**: lazily created `AudioContext` (`ensureAudio()`), with `beep()`, `noiseBurst()`, and a `sfx` map. Audio must start from a user gesture.
7. **Rendering helpers**: `drawPlane()`, `drawBossShip()`, `drawClouds()`, and `draw()`.
8. **Spawning and combat**: `spawnEnemy()`, `spawnBoss()`, `spawnExplosion()`, `fireWeapon()`, and `update(dt)`.
9. **Boss patterns**: `fireRadial`, `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`, `fireCross`, `fireHomingSeeded`, `fireDoubleRing`, driven by `updateBoss(dt)`.
10. **Game loop**: `loop(now)` clamps `dt` to 0.05 s, then calls `update` and `draw` when running and not paused. It is driven by `requestAnimationFrame`.
11. **Input wiring**: keyboard listeners (`keydown` / `keyup`), mouse and touch handlers, the on-screen joystick and FIRE / BOMB buttons, and the overlay buttons.

## Controls (keep the hint text in sync)

The on-screen hint at `#controlsHint` documents these. If you change a binding, update that text too.

- Mouse: move to steer, click to fire.
- Keyboard: Arrows or WASD to move, Space to fire, X or Shift for a bomb, P or Esc to pause.
- Touch: drag on the left to fly, tap FIRE, tap BOMB.

## Conventions

- **Single file, vanilla JS.** Keep new code inside the existing IIFE and follow its style: 2-space indent, `const` by default, camelCase names, and `UPPER_SNAKE` for config constants.
- **Tune through the config tables.** Put balance numbers in `DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, or `THEMES` rather than hard-coding them in logic.
- **Persist settings through `localStorage`** with a `skyfighter.`-prefixed key. Wrap access in try/catch if you add new reads or writes, because storage can be unavailable (for example in private modes).
- **Respect reduced motion.** Effects such as screen shake, hit flash, and the low-health pulse are gated by `reducedMotion`. New motion effects should be gated the same way.
- **Accessibility.** Announce important state changes through `announce()`. Keep buttons labelled with `title` or text.
- **Mobile.** The layout uses `safe-area-inset` and `touch-action: none`. Test touch controls whenever you change layout or input code.
- **Pause and reset.** `togglePause()` and `resetGame()` own the run lifecycle. New timers or state must be cleared in `resetGame()`.

## Git

- Development branch for this work: `claude/gifted-tesla-0s3m0w`. Push there unless told otherwise.
- Do not open pull requests unless explicitly requested.
