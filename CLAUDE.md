# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a small static web project. Its only real content is **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs as a single-page HTML5 canvas game.

| File | Purpose |
| --- | --- |
| `index.html` | The entire game: markup, CSS, and JavaScript in one self-contained file (~1,670 lines). |
| `README.md` | One-line title only. |

There is no build step, package manager, bundler, test suite, linter, or CI configuration. Do not add tooling unless the user asks for it.

## Running and verifying

- Run the game by opening `index.html` in a browser. No server is required.
- Since there are no automated tests, verify changes by loading the page and playing: start a game, move, fire, use a bomb, pause/resume, die, and check the game-over card. Test at both desktop and phone-sized viewports, because touch controls appear on touch devices.
- For visual or behavioral checks in a headless environment, Playwright with the preinstalled Chromium works (see the environment notes), but it is not wired into the repo.

## Architecture of `index.html`

The file is organized top to bottom as follows. Keep new code in the matching section.

1. **`<head>`**: viewport meta (`user-scalable=no`, `viewport-fit=cover`), theme color, and title.
2. **`<style>`** (around lines 10–359): all styling. HUD is a pill-style bar with heart lives and bomb icons; the overlay is a glassmorphic card; touch controls are shown only when `.enabled` is applied to `#touchControls`.
3. **Markup** (around lines 362–420): `#container` holds the `<canvas id="gameCanvas">`, screen-flash and vignette layers, the HUD, the touch controls, the overlay (title, difficulty buttons, start button), and the live-region `#srLive` for screen-reader announcements.
4. **`<script>`** (around line 422 onward): a single IIFE. Inside it, in order:
   - DOM lookups as `const` (`canvas`, `ctx`, `scoreEl`, …). Add new element references here rather than calling `getElementById` inside loops.
   - `STORAGE_*` key constants and persisted settings (best score, mute, difficulty, reduced motion).
   - Audio: a lazily created `AudioContext` (`ensureAudio`), with `beep`, `noiseBurst`, and a named `sfx` table. Audio starts only after a user gesture.
   - HUD and effect helpers: `renderHearts`, `renderBombs`, `flashHit`, `showComboPop`, `showStageBanner`, `screenShake`.
   - Tuning tables: `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES` (one per stage, cycled), `ENEMY_TYPES`, `BOSSES`.
   - Input: keyboard (`keys` map keyed by `e.code`), touch joystick and buttons, and mouse aiming on the canvas.
   - Game state (`let` variables) and `resetGame()`.
   - Drawing: `drawPlane`, `drawBossShip`, background and clouds, `draw()`.
   - Simulation: `spawnEnemy`, `spawnBoss`, `spawnExplosion`, `fireWeapon`, `update(dt)`, boss attack patterns (`fireRadial`, `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`, `fireCross`, `fireHomingSeeded`, `fireDoubleRing`), `updateBoss`, `endGame`.
   - `loop(now)` drives everything through `requestAnimationFrame`. `dt` is clamped to 0.05 s so tab switches don't cause large jumps.

## Game design reference

- **Difficulty** (`DIFFICULTIES`): easy / normal / hard change spawn rates, enemy speed, enemy fire chance, starting lives and bombs, and boss HP multiplier.
- **Stages**: each stage requires `KILLS_PER_STAGE` (18) kills, then a boss fight. The boss and theme are chosen as `BOSSES[(stage - 1) % BOSSES.length]` and `THEMES[(stage - 1) % THEMES.length]`, so both cycle.
- **Enemies**: `grunt` (1 HP), `interceptor` (weaving, fast, 1 HP), `tank` (4 HP, appears from stage 2).
- **Weapons**: Vulcan and Laser, each with levels 1–3. A `weapon` pickup raises the level; at level 3 it switches to the other weapon and resets to level 1.
- **Power-ups**: `life`, `bomb`, and `weapon`.
- **Scoring**: a kill-chain combo sets `multiplier`; combo pops are shown on the HUD.
- **Persistence**: `localStorage` keys are prefixed `skyfighter.` (`bestScore`, `muted`, `difficulty`, `reducedMotion`).

## Controls

- Move: arrow keys or WASD. Mouse moves the ship over the canvas and holding the mouse button on the canvas fires.
- Fire: Space (hold), mouse hold on the canvas, or the FIRE button.
- Bomb: X or Shift, or the BOMB button.
- Pause: P or Escape, or the pause button.
- Touch: left-side joystick, right-side fire and bomb buttons.

## Conventions

- **Vanilla JS and CSS only.** No frameworks, no external scripts or fonts. Keep the game self-contained in `index.html`.
- **Match the existing style**: two-space indentation, single quotes in JS, `const` by default, short arrow functions for listeners, and section-style organization within the IIFE.
- **Tune through the tables**, not by scattering magic numbers. Difficulty, enemy stats, boss stats, and themes belong in the constant objects near the top of the script.
- **Accessibility**: announce state changes through `announce()` (which writes to `#srLive`). Respect `prefers-reduced-motion`; the `reducedMotion` flag gates screen shake, hit flashes, and the low-health pulse. New screen-shake or flash effects must check that flag.
- **Mute and reduced-motion toggles** must persist via their `STORAGE_*` keys and update their button state through the existing `update*Btn` helpers.
- **Performance**: the game self-downgrades through `quality` (0 full, 1 medium, 2 low) based on frame timing in `trackPerf`. Effects added to `draw()` should respect that tier.
- **Guard browser APIs**: `localStorage` reads/writes can fail in private modes, so wrap new storage access defensively if it can throw.
- **Keep the mobile layout working.** Test any HUD or overlay change at phone widths. `resize()` scales the 480×720 base canvas to fit the window (capped at 1.6×).

## Git workflow

- The default branch is `main`. Feature work goes on a dedicated `claude/...` branch that is pushed with `git push -u origin <branch>`.
- Write clear, descriptive commit messages in the imperative mood, with a summary line and a short body for non-trivial changes.
- Do not open pull requests unless the user asks for one.
- Do not commit generated or local-only files.

## When making changes

- Read the relevant section of `index.html` before editing; the file is large, so search for the function or constant first instead of reading it whole.
- Keep changes focused. This project has no test net, so review the diff carefully and play-test the affected behavior before reporting it done.
- If a change touches saved data keys, difficulty numbers, or boss patterns, call that out in the commit message so players' persisted state and balance expectations are understood.
