# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a small, dependency-free browser game, **Sky Fighter**: a Raiden-style vertical shoot-'em-up. The entire game lives in one self-contained file, `index.html` (inline `<style>` and an inline `<script>`, drawn on a 2D `<canvas>`). There is no build step, no package manager, and no test suite.

Files:

- `index.html` — the whole game (markup, CSS, and JavaScript).
- `README.md` — a one-line placeholder. Keep it accurate if you expand it.
- `CLAUDE.md` — this file.

## Running and testing

- Open `index.html` directly in a browser, or serve the directory with any static server (for example `python3 -m http.server`).
- There are no automated tests, linters, or build commands. Verify changes by playing the game in a browser: start a run, check firing/movement, bombs, pause (`P`/`Esc`), game over, and the high score.
- To verify on touch layouts, use a mobile-sized viewport; touch controls only appear when the device reports touch support.

## Code structure (inside `index.html`)

All game logic is wrapped in a single IIFE with `'use strict'`. Nothing is exposed on `window`. Keep new code inside that scope.

Major sections, in file order (search for the `// ----------` comment banners and function names):

- **DOM references** — `getElementById` lookups at the top of the script.
- **Storage keys** — `STORAGE_BEST`, `STORAGE_MUTE`, `STORAGE_DIFF`, `STORAGE_REDUCED` (all prefixed `skyfighter.`).
- **Adaptive performance** — `trackPerf` samples frame times and steps a `quality` level (0 full, 1 medium, 2 low) to shed effects when frame rate drops.
- **Tuning data** — constant tables that drive balance:
  - `DIFFICULTIES` (`easy` / `normal` / `hard`): spawn rates, enemy speed, shot chance, starting lives and bombs, boss HP multiplier.
  - `KILLS_PER_STAGE` (18) — kills needed to advance a stage.
  - `THEMES` — six background palettes that cycle with stages.
  - `ENEMY_TYPES` — `grunt`, `interceptor` (weaving), `tank` (4 HP, appears from stage 2).
  - `BOSSES` — six bosses (VANGUARD, BEHEMOTH, SPECTER, DREADNOUGHT, WRAITH, TITAN), each with colours and an `hpMul`.
  - `BOSS_PATTERNS` — bullet patterns (radial, aimed spread, columns, spiral, cross, homing, double ring).
- **Audio** — `ensureAudio` lazily creates the `AudioContext` (browsers require a user gesture first). `beep` and `noiseBurst` synthesize all sound effects; there are no audio files. `sfx.*` is the public entry point.
- **HUD and feedback** — hearts, bombs, combo pop-ups, stage banner, screen shake, hit/bomb flashes, and an `aria-live` region (`announce`) for screen readers.
- **Input** — keyboard (`keys` map keyed by `e.code`), touch joystick plus fire/bomb buttons, and mouse (`mousemove` / `mousedown` on the canvas).
- **Layout** — `BASE_W` × `BASE_H` is 480 × 720. `resize()` scales the canvas to fit the window, capped at 1.6×.
- **Game state and loop** — `resetGame`, `update(dt)`, `draw()`, and `loop(now)`. `dt` is clamped to 50 ms. Pause is handled by `togglePause`; game over by `endGame` / `loseLife`.
- **Power-ups** — `applyPowerUp` handles `life`, `bomb`, and `weapon`. Weapons are `vulcan` and `laser`, each with three levels; picking up a weapon power-up past level 3 switches weapon type.

Conventions observed in the existing code:

- Use `const` for tuning tables and `let` for mutable state; the file uses 2-space indentation and single quotes.
- Drawing helpers are prefixed `draw`; spawning helpers are prefixed `spawn`; state transitions use verbs such as `resetGame`, `endGame`, `killBoss`.
- Gameplay values are plain numbers in the tuning tables. Prefer adding a value to a table over hard-coding it in a function.

## Persistence

The game stores four values in `localStorage`, all with the `skyfighter.` prefix:

| Key | Meaning |
| --- | --- |
| `skyfighter.bestScore` | High score |
| `skyfighter.muted` | `'1'` when sound is muted |
| `skyfighter.difficulty` | `easy`, `normal`, or `hard` |
| `skyfighter.reducedMotion` | `'1'` / `'0'`; defaults to the `prefers-reduced-motion` media query when unset |

Storage access is not wrapped in `try/catch`. If you add storage, keep the same prefix and consider failure modes such as private browsing.

## Accessibility and motion

- Reduced motion is applied by toggling the `reduced-motion` class on `<body>`; CSS and effects respect it. Any new animation should be checked against this mode.
- Status changes should be announced through `announce()`, not by writing to the DOM directly.

## Editing guidelines

- Keep the game a single file unless there is a clear reason to split it. Do not add a build step or dependencies without asking.
- When changing balance, edit the tuning tables (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `KILLS_PER_STAGE`) rather than scattering magic numbers.
- Preserve the existing `quality` level handling when adding particles or effects; heavy effects should be reduced at higher quality levels.
- Verify every change in a browser, including at least one full run through a stage boss, before reporting it done.
- Do not commit generated or temporary files to the repository.

## Git

- Default development branch for this workspace: `claude/gifted-tesla-ir9z0j`.
- Commit messages should be short and descriptive in the imperative mood (the existing history uses e.g. "Add Sky Fighter: Raiden-style vertical shoot-em-up game").
