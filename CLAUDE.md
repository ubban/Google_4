# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a small static web project. Its only real content is **Sky Fighter**, a Raiden-style vertical shoot-'em-up game, delivered as a single self-contained HTML5 canvas page.

- `index.html` (~58 KB): the entire game — markup, CSS and JavaScript inline. No build step, no dependencies, no package manager.
- `README.md`: placeholder title only.
- No tests, linter, CI config or `package.json` exist.

## Repository layout

```
index.html   # Whole game: <style> (~lines 10-420), HTML body, <script> (~lines 422-end)
README.md
CLAUDE.md    # this file
```

## Running it

Open `index.html` directly in a browser, or serve the directory with any static server, e.g. `python3 -m http.server` and visit `http://localhost:8000`. There is nothing to install or compile.

## Architecture of `index.html`

The script is one IIFE (`'use strict'`) that wires DOM elements to a `requestAnimationFrame` loop (`loop` → `update(dt)` → `draw()`).

Key sections, in source order:

- **DOM refs and storage keys** — `STORAGE_*` constants use the `skyfighter.*` prefix (`bestScore`, `muted`, `difficulty`, `reducedMotion`).
- **Adaptive quality** — `trackPerf` samples frame times and steps `quality` (0 full / 1 medium / 2 low).
- **Tuning data** — `DIFFICULTIES` (spawn rates, enemy speed, shot chance, lives, bombs, boss HP multiplier), `KILLS_PER_STAGE`, `THEMES` (per-stage sky palette), `ENEMY_TYPES` (grunt / interceptor / tank), `BOSSES` (six bosses, cycled by stage).
- **Audio** — lazily created Web Audio context (`ensureAudio`), with `beep` and `noiseBurst` helpers and per-enemy explosion SFX. Respects the mute flag.
- **Input** — keyboard (`keys` map keyed by `KeyboardEvent.code`), mouse/pointer steering and click-to-fire, and touch joystick plus FIRE/BOMB buttons.
- **Game state** — `resetGame()` initialises everything; `update(dt)` advances the simulation; `draw()` renders it.
- **Weapons and power-ups** — `fireWeapon()` (Vulcan/Laser, 3 levels), `applyPowerUp()`, `maybeSpawnPowerUp()`, `useBomb()`.
- **Bosses** — `BOSS_PATTERNS` are bullet-pattern functions (radial, spread, spiral, cross, homing, double ring); `updateBoss` / `killBoss` manage them.
- **Scoring** — `addScore()` applies a kill-chain multiplier; `breakChain()` resets it.
- **Lifecycle** — `loseLife()`, `endGame()`, `togglePause()`.

Conventions in this file:

- Sizes are authored at a base resolution of `BASE_W = 480`, `BASE_H = 720`, then scaled by `resize()`.
- Entities are plain objects in arrays; collisions use `rectsOverlap`.
- Accessibility: `announce()` writes to an ARIA live region; `reducedMotion` (user toggle, defaults to `prefers-reduced-motion`) scales down screen shake and hit flashes.

## Controls (for manual testing)

- Keyboard: arrows / WASD move, Space fire, X or Shift bomb, P or Esc pause.
- Mouse: move to steer, click to fire.
- Touch: drag left side to fly, tap FIRE, tap BOMB.

## Persistence

Only `localStorage` is used, under keys prefixed `skyfighter.`. Every access must tolerate storage being unavailable (private mode, blocked site data), so keep reads and writes defensive when changing them.

## Editing guidelines

- Keep the game as a single file unless the user asks for a split. Do not add a build step, framework or npm dependencies without a clear reason.
- Preserve the single IIFE and `'use strict'`. Don't introduce globals.
- Put new tuning numbers in the existing data tables (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `THEMES`) rather than hard-coding them in logic.
- Keep the layout working at phone width (`viewport-fit=cover`, safe-area insets, `touch-action: none`) and avoid horizontal page scroll.
- Keep the page free of external network requests; the favicon is an inline SVG data URI and fonts fall back to system stacks.
- If you change the controls, update the `#controlsHint` text in the markup as well.

## Verification

There is no automated test suite. To verify a change:

1. Open `index.html` in a browser (or use Playwright with the pre-installed Chromium) and check the console for errors.
2. Play through at least one stage, exercise weapons, a bomb, a power-up and the boss, and confirm the HUD updates.
3. Check the mobile layout at a narrow viewport and confirm the touch controls appear.
4. Confirm that the best score, mute, difficulty and reduced-motion settings survive a reload.

## Git

- Development branch for this session: `claude/gifted-tesla-317om1`.
- Commit messages are short imperative subjects, with a body for larger changes (see `git log`).
