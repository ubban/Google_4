# CLAUDE.md

Guidance for Claude Code when working in this repository.

## What this repo is

`Google_4` is a single-page browser game, **Sky Fighter** — a Raiden-style vertical shoot-'em-up. The entire game is one self-contained file:

- `index.html` — HTML, CSS (inline `<style>`), and JavaScript (inline `<script>`, wrapped in an IIFE with `'use strict'`) all live here. Rendering is done on a single `<canvas id="gameCanvas">`.
- `README.md` — a one-line title only.

There is no build step, no package manager, no bundler, no framework, no test suite, and no linter configured. Nothing is installed or compiled.

## Running and verifying

- **Run:** open `index.html` directly in a browser (double-click or `file://`), or serve the directory with any static server (e.g. `python3 -m http.server`) and browse to it.
- **Verify changes:** there are no automated tests. Load the page in a browser, start a game, and exercise the change manually. For UI or gameplay changes, check desktop (keyboard/mouse) and, if relevant, touch controls.
- **Syntax sanity check** (optional, quick): extract the `<script>` block and run `node --check` on it, e.g. with a short one-off script in the scratchpad directory. Do not commit that helper script to the repo.

## Code layout inside `index.html`

Roughly in order within the `<script>` block:

1. **DOM refs and persistence keys** — `STORAGE_BEST`, `STORAGE_MUTE`, `STORAGE_DIFF`, `STORAGE_REDUCED` (all under the `skyfighter.*` prefix in `localStorage`).
2. **Performance scaling** — `trackPerf()` adjusts a `quality` level (0 full / 1 medium / 2 low) from a rolling frame-time average.
3. **Tuning tables** — `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES` (sky palettes per stage), `ENEMY_TYPES` (`grunt`, `interceptor`, `tank`), `BOSSES` (six boss designs, cycled by stage).
4. **Audio** — a lazily created `AudioContext` (`ensureAudio()`), with `beep()` / `noiseBurst()` helpers and a named `sfx` object.
5. **Input** — keyboard (`keys[e.code]`), mouse on the canvas, and touch joystick + FIRE/BOMB buttons.
6. **Game state and loop** — state variables (`score`, `lives`, `bombCount`, `stage`, `kills`, `chain`, `multiplier`, etc.), `resetGame()`, `update(dt)`, `draw()`, and the `requestAnimationFrame` loop `loop(now)`. `dt` is clamped to 0.05 s.
7. **Gameplay systems** — spawning (`spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`), weapons (`fireWeapon`, `player.weapon` is `vulcan` or `laser`), power-ups (`applyPowerUp`: `life`, `bomb`, `weapon`), bombs (`useBomb`), scoring/chain/multiplier (`addScore`, `breakChain`), and boss attack patterns (`BOSS_PATTERNS`, `updateBoss`, `killBoss`).
8. **Rendering** — `drawPlane`, `drawBossShip`, `drawClouds`, `draw()`, plus screen shake and vignette/flash effects.
9. **Overlay / UI wiring** — start button, difficulty buttons, mute and reduced-motion toggles, pause (`togglePause`), game over (`endGame`), and the HUD (hearts, bombs, score, stage, weapon, multiplier).

Keep new code in the matching section rather than adding a new top-level block.

## Conventions

- **Single file.** Do not split `index.html` into separate JS/CSS files or add a build step unless the user asks for it.
- **No dependencies.** Use plain HTML/CSS/JS and the Canvas 2D and Web Audio APIs already in use. Do not add CDN scripts or libraries without an explicit request.
- **Style:** match the existing code — 2-space indentation, single quotes in JS, `const`/`let` (no `var`), arrow functions for callbacks, and the `// ---------- Section ----------` comment banners.
- **Tuning changes** belong in the constant tables (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `THEMES`) rather than hard-coded in logic where practical.
- **Accessibility:** keep the `srLive` announcements (`announce()`) and the `reducedMotion` setting working when adding visual effects — effects that shake, flash, or pulse should respect the reduced-motion toggle.
- **Mobile:** the canvas scales to fit the viewport (`resize()`, base resolution 480×720). Keep touch controls (`#touchControls`) working when changing layout or input.
- **Persistence:** wrap any new `localStorage` access with the same pattern as the existing keys, and use the `skyfighter.` prefix for new keys.

## Gameplay reference (for changes that touch balance or controls)

- **Controls:** mouse to steer and click to fire; Arrows/WASD to move; Space to fire; X or Shift for a bomb; P or Esc to pause. Touch: drag left side to fly, FIRE and BOMB buttons.
- **Difficulty:** Easy / Normal / Hard change spawn rate, enemy speed, enemy fire chance, starting lives, starting bombs, and boss HP multiplier.
- **Stages:** each stage requires `KILLS_PER_STAGE` (18) kills before a boss fight; stages advance the theme and boss.
- **Scoring:** kills award points with a chain multiplier that breaks when the chain timer expires.

## Git and workflow

- Develop on the branch designated for the task (currently `claude/gifted-tesla-1t644j`). Do not push to other branches without explicit permission.
- Commit with clear, descriptive messages. Push with `git push -u origin <branch>`.
- Do not open a pull request unless the user explicitly asks for one.
- Keep commits focused: one logical change per commit where practical.

## Updating this file

Update `CLAUDE.md` when the layout, conventions, or gameplay reference above changes meaningfully (for example, a new file is added, a new weapon or enemy type is introduced, or the control scheme changes). Keep it factual and short; do not paste large chunks of code into it.
