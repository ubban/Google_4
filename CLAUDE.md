# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo is **Sky Fighter**, a Raiden-style vertical shoot-em-up built as a single self-contained HTML5 canvas page.

- `index.html` — the entire game: markup, CSS (`<style>`, ~lines 10–360), and JavaScript (`<script>`, ~line 422 onward). No build step, no bundler, no external dependencies.
- `README.md` — placeholder; currently contains only a title and a stray timestamp line.

There is no package manager, test suite, linter, or CI configuration in the repo.

## Running and verifying

- Open `index.html` directly in a browser, or serve the directory with any static server (e.g. `python3 -m http.server`).
- There are no automated tests. Verify changes by playing the game in a browser, covering at least: start, movement, firing, bomb, pause/resume, a stage transition, a boss fight, game over, and the restart flow.
- Test both desktop (keyboard/mouse) and touch layouts when touching input or HUD code.

## Code layout inside `index.html`

The script is wrapped in a single IIFE (`'use strict'`). Everything is module-private; nothing is exported to `window`.

Key sections, roughly in file order:

- **DOM references** — cached `getElementById` lookups at the top of the IIFE.
- **Storage keys** — `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion` in `localStorage`.
- **Adaptive performance** — `trackPerf()` samples frame times and steps a `quality` level (0–2) up or down.
- **Tuning tables**
  - `DIFFICULTIES` — `easy` / `normal` / `hard`: spawn rates, enemy speed, enemy shoot chance, starting lives and bombs, boss HP multiplier.
  - `KILLS_PER_STAGE` — kills needed before a boss wave (currently 18).
  - `THEMES` — six sky palettes, cycled by stage.
  - `ENEMY_TYPES` — `grunt`, `interceptor` (weaving), `tank` (4 HP, unlocks at stage 2).
  - `BOSSES` — six boss designs (`VANGUARD` … `TITAN`), chosen by `(stage - 1) % length`, each with an `hpMul`.
- **Audio** — lazily created `AudioContext` (`ensureAudio`), with `beep()` and `noiseBurst()` helpers. Sound is synthesized; no audio files.
- **Input** — keyboard (`keys[e.code]`), a virtual joystick and fire/bomb buttons for touch, and mouse/pointer support on the canvas.
- **Game state** — top-level `let` variables (score, lives, bombs, stage, chain/multiplier, etc.) reset by `resetGame()`.
- **Weapons and bosses** — `fireWeapon()` handles the Vulcan/Laser weapon with three power levels. `BOSS_PATTERNS` holds bullet-pattern functions (radial, aimed spread, columns, spiral, cross, homing, double ring); `updateBoss()` chooses among them.
- **Update / draw** — `update(dt)` advances the simulation and `draw()` renders it. `loop(now)` runs each frame, clamps `dt` to 50 ms, and calls `update` and `draw` only while `running && !paused`.

### Controls (for reference when changing input)

| Action | Keyboard | Mouse | Touch |
|---|---|---|---|
| Move | Arrow keys or WASD | Move pointer over canvas | Drag left zone (virtual joystick) |
| Fire | Hold Space | Hold mouse button | Hold FIRE button |
| Bomb | X or Shift | — | BOMB button |
| Pause | P or Escape | — | Pause button |

Mute is a button (`#muteBtn`) with no keyboard shortcut.

Keyboard input is read via `e.code`, so layout-independent codes such as `KeyW` and `ShiftLeft` are the correct identifiers.

## Conventions

- Keep the game in the single `index.html` file unless the user asks for a split. Do not introduce a build tool or framework without being asked.
- Keep the IIFE and `'use strict'`. Do not leak globals.
- Use `const` for tuning tables and `let` only for mutable state, matching the existing style.
- Preserve the `localStorage` access pattern. Storage calls can throw in some browser privacy modes, so wrap new storage access in `try/catch` if you add any.
- Accessibility features are intentional: keep the `aria-live` announcer (`announce()`), the reduced-motion setting, and the keyboard-operable controls working when you change UI.
- Keep the visual style (pill HUD, glass overlay card, combo pop-ups, screen shake) consistent with the existing CSS.

## Making changes

- Tuning changes (speeds, spawn rates, HP, scores, bullet counts) belong in the constant tables near the top of the script. Prefer editing those over hard-coding new numbers in logic.
- When adding a boss pattern, add a function and append it to `BOSS_PATTERNS` so the existing selection logic picks it up.
- When adding a power-up, extend the random roll in the spawn code and handle the new `type` in `applyPowerUp()`.
- Before committing, play-test the affected path in a browser. There is no automated check to rely on.

## Git workflow

- Develop on the branch the task specifies (currently `claude/gifted-tesla-21bz6y`), and do not push to other branches without permission.
- Use descriptive commit messages that explain the gameplay or code change.
- Do not open a pull request unless explicitly asked.
