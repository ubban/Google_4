# CLAUDE.md

Guidance for Claude Code when working in this repository.

## What this repo is

`Google_4` is a single-page browser game, **Sky Fighter** — a Raiden-style vertical shoot-'em-up. Everything lives in one file:

- `index.html` (~1,670 lines): markup, inline `<style>`, and one inline `<script>` wrapped in an IIFE with `'use strict'`. No external assets, libraries, or network calls.
- `README.md`: a stub title with a stray timestamp line. It is not user documentation.

There is no `package.json`, bundler, linter, test suite, or CI config. Do not invent build or test commands. To run the game, open `index.html` in a browser (or serve the directory with any static server).

## Layout of `index.html`

Markup and CSS come first (the `<style>` block runs lines 10–359; the canvas and HUD/overlay markup follow it). The script block begins at line 422. Within the script, the sections appear in this order:

1. **DOM references and storage keys** — `STORAGE_*` constants with the `skyfighter.*` prefix.
2. **Accessibility/perf helpers** — `announce()` (writes to the `#srLive` aria-live region) and `trackPerf()`, which steps a `quality` level (0–2) from frame times.
3. **Tuning tables** — `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES` (sky palettes per stage), `ENEMY_TYPES`, `BOSSES`. Most balance changes belong here.
4. **Audio** — `ensureAudio()` lazily creates the `AudioContext` (browsers require a user gesture), plus `beep`, `noiseBurst`, and the `sfx` helpers.
5. **HUD and effects** — `renderHearts`, `renderBombs`, banners, screen shake, flashes.
6. **Canvas sizing** — `BASE_W = 480`, `BASE_H = 720`; `resize()` scales the canvas to fit the window, capped at 1.6× the base size.
7. **Input** — keyboard (`keydown`/`keyup` on `e.code`), mouse on the canvas, and touch joystick/buttons.
8. **Game state and update** — `resetGame()`, `update(dt)`, spawning, scoring/chain (`addScore`, `breakChain`), power-ups, bombs, `loseLife()`, `endGame()`, `togglePause()`.
9. **Bosses** — `BOSS_PATTERNS` (bullet-pattern functions such as radial, spiral, and homing), `updateBoss`, `killBoss`.
10. **Rendering** — `draw()` and the sprite helpers (`drawPlane`, `drawBossShip`, `drawClouds`).
11. **Main loop** — `loop(now)` runs on `requestAnimationFrame` and clamps `dt` to 50 ms.

## Controls (keep the on-screen hint in sync)

The `#controlsHint` text in the markup lists the controls. If you change any binding, update that string too.

- Mouse: move to steer, click to fire. Keyboard: Arrows/WASD move, Space fires, X or Shift drops a bomb, P or Esc pauses.
- Touch: drag left to fly, tap FIRE, tap BOMB.

## Persistence

Settings are stored in `localStorage` under `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, and `skyfighter.reducedMotion`. Any new persisted value should use the same `skyfighter.` prefix and be read with a fallback default, since the stored value may be missing.

## Conventions

- **Single file.** Keep the game in `index.html`. Do not add a build step, framework, or external dependency unless the user asks for one.
- **Style.** Match the existing code: 2-space indent, `const`/`let`, arrow functions for callbacks, and section banners written as `// ---------- Name ----------`.
- **Tuning changes.** Put numeric balance values in the existing tables rather than hard-coding them in update or draw logic.
- **Frame-rate independence.** Movement and timers use `dt` (seconds). Preserve that when adding behavior.
- **Accessibility.** Respect the `reduced-motion` class and the `reducedMotion` flag when adding screen shake, flashes, or other motion effects. Announce important state changes through `announce()`.
- **Mobile.** Keep the layout usable at phone widths and make sure new controls have touch equivalents.

## Verifying changes

Nothing is automated, so verify by hand:

1. Open `index.html` in a desktop browser and check the console for errors.
2. Play through at least one stage, a boss fight, a bomb, a power-up, and a game over, on each difficulty if the change touches balance.
3. Check touch controls with browser device emulation if the change touches input or layout.
4. Confirm the best score survives a reload and that the mute and reduced-motion toggles persist.

If you add a test or build tool, document its commands in this file.

## Git

- The default branch is not the development branch. Work on the branch you were given (`claude/gifted-tesla-u4y2q3` for this session) and push there.
- Do not open a pull request unless the user asks for one.
