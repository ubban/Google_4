# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a small static web project. Its only real content is **Sky Fighter**, a Raiden-style vertical shoot-'em-up written as a single self-contained HTML5 canvas page.

| File        | Purpose                                                                 |
|-------------|-------------------------------------------------------------------------|
| `index.html` | The entire game: markup (~360 lines), inline CSS, and inline JavaScript (~1,670 lines total). |
| `README.md`  | Placeholder heading only. No setup or usage docs yet.                  |
| `CLAUDE.md`  | This file.                                                              |

There is no build step, package manager, bundler, test suite, linter, or CI configuration. Do not invent commands for those. To run the game, open `index.html` in a browser (or serve the directory with any static server).

## Architecture of `index.html`

The file is organized top to bottom as: `<head>` meta and favicon (inline SVG data URI) → inline `<style>` → HUD / overlay / touch-control markup → one `<script>` block wrapped in an IIFE.

Key sections of the script, in source order (approximate line numbers, will drift):

- **Storage keys** (~line 463): `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion`. These are the only persisted state. Keep these names stable, since changing them resets players' saved settings and best score.
- **Config tables** (~lines 485–525): `DIFFICULTIES` (easy / normal / hard: spawn rates, enemy speed and fire chance, lives, bombs, boss HP multiplier), `KILLS_PER_STAGE` (18), `THEMES` (six sky palettes that cycle per stage), `ENEMY_TYPES` (grunt, interceptor, tank), `BOSSES` (six named bosses with colors and HP multipliers).
- **Audio** (~line 545 onward): a lazily created `AudioContext` (`ensureAudio`) with `beep` and `noiseBurst` helpers. Sound only starts after a user gesture.
- **Input** (~lines 769–870): keyboard (`keys[e.code]`), a touch joystick on the left zone with FIRE and BOMB buttons, and mouse steering and firing on the canvas.
- **Game state and loop**: `resetGame()` (~line 875) initializes state. `update(dt)` (~line 1148) is the per-frame simulation. Rendering helpers (`drawPlane`, `drawBossShip`) sit above it.
- **Weapons and bombs**: `fireWeapon`, `useBomb`, `applyPowerUp`. Vulcan and Laser weapons, each with three power levels.
- **Bosses**: `spawnBoss`, `updateBoss`, and the `BOSS_PATTERNS` array of bullet-pattern functions (radial, aimed spread, columns, spiral, cross, homing, double ring).
- **Presentation**: HUD updates (`renderHearts`, `renderBombs`), screen shake, hit flash, combo popups, stage banners, and a `reducedMotion` toggle that disables shake, flash, and pulse.
- **Responsive sizing**: `resize()` fits a fixed logical canvas of `BASE_W × BASE_H` = 480 × 720 into the viewport.

When changing game balance, edit the config tables rather than hard-coding numbers in `update()`.

## Conventions

- **Keep it one file.** The game is deliberately a single HTML file with no external dependencies. Do not add a bundler, framework, or external asset without asking first. If a new file is needed, discuss it first.
- **Preserve the IIFE.** Everything runs inside `(function () { ... })()`. Do not leak globals.
- **Use `const` for config, `let` for state.** Follow the existing style of 2-space indentation and single quotes in JS.
- **Mobile and desktop both matter.** Any new input must work with keyboard, mouse, and touch. The touch controls are hidden on desktop and shown on touch devices.
- **Respect reduced motion.** Any new screen-shake, flashing, or pulsing effect must check `reducedMotion`.
- **Accessibility.** Announce important state changes through the `#srLive` live region (see the `announce()` helper).
- **Preserve `prefers-reduced-motion` and `user-scalable=no` viewport behavior** unless the user asks to change them.

## Working on this repo

- Develop on the branch `claude/gifted-tesla-1xcpdc`. Do not push to other branches without explicit permission.
- Do not create a pull request unless the user explicitly asks for one.
- Before committing, open `index.html` in a browser or a headless Chromium (Playwright is pre-installed in the remote environment) and confirm the game starts, runs, and pauses. There are no automated tests to rely on.
- Keep `README.md` and this file in sync when the project's shape changes (for example, when a second file or a build step is added).
- Commit messages should be short and imperative, matching the existing history (for example, `Add Sky Fighter: ...`).

## Things to know before changing code

- The loop is variable-step: `dt` is derived from `requestAnimationFrame` timestamps and clamped to 0.05 s (~line 1642). Gameplay values are per-frame-scaled by `dt`, so check changes at both 60 Hz and a lower frame rate.
- Spawns and boss patterns use `Math.random()` directly; there is no seeded RNG.
- The `localStorage` reads at the top of the script (~lines 463–475) are not wrapped in try/catch, and `localStorage` throws in some privacy modes. Guard any new storage access, and consider guarding the existing reads if you touch that code.
- Touch controls are enabled at runtime by feature detection (`ontouchstart` or `navigator.maxTouchPoints`, ~line 778), not by CSS media queries.
- Canvas coordinates are logical (480 × 720). Do not use CSS pixel sizes for game logic.
