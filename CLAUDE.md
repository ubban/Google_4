# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a small, dependency-free browser game called **Sky Fighter**, a Raiden-style vertical shoot-'em-up. The entire game lives in a single file, `index.html` (~1,670 lines: inline CSS, markup, and a single `<script>` block). `README.md` is a placeholder.

There is no build step, package manager, bundler, framework, test suite, or lint configuration. Do not add one unless asked.

## Running the game

Open `index.html` directly in a browser, or serve the directory with any static server (for example `python3 -m http.server`). Nothing needs to be installed.

## Verifying changes

There are no automated tests. After changing game logic or UI:

1. Load the page and start a game from the overlay (Normal difficulty by default).
2. Exercise the path you changed: movement, firing, bombs, pause, a stage transition, the boss fight, and game over.
3. Check the browser console for errors.
4. If the change touches layout, check both a desktop viewport and a narrow mobile viewport, since the canvas scales to fit and touch controls appear on touch devices.

Use the `run` skill if you need a repeatable way to launch and screenshot the app.

## Architecture

Everything is inside `index.html`. The file is organized in this order:

- **`<style>`**: HUD, overlay card, touch controls, and the reduced-motion and animation rules. Colors and layout are plain CSS; keep it that way.
- **Markup**: `#container` holds the `<canvas id="gameCanvas">`, the HUD (`#hud`), the boss bar, the top bar (mute, reduced-motion, pause), touch controls, and the start/game-over `#overlay`.
- **Script** (wrapped in an IIFE-style block; globals are not leaked):
  - **DOM references** at the top, one `const` per element.
  - **Config tables**: `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES` (one per stage, cycled), `ENEMY_TYPES` (with `minStage` gating), `BOSSES` (cycled, with an `hpMul`).
  - **Persistence**: `localStorage` for best score, mute, difficulty, and reduced motion (see keys below).
  - **Audio**: a WebAudio synth (`ensureAudio`, `beep`, `noiseBurst`, the `sfx` map). There are no audio files. The `AudioContext` is created lazily on a user gesture.
  - **Input**: keyboard (`keys[]`), mouse (`mouseTarget`, click to fire), and touch (`joystick*`, fire and bomb buttons).
  - **Game state**: `player`, `bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`, `boss` and a set of scalars (`score`, `lives`, `bombCount`, `stage`, `kills`, `chain`, `multiplier`, and so on). `resetGame()` initializes all of it.
  - **Simulation**: `update(dt)` advances the world. `updateBoss(dt)` drives boss phases and the `fire*` pattern helpers (`fireRadial`, `fireAimedSpread`, `fireCross`, and others).
  - **Rendering**: `draw()` renders the frame. `drawPlane`, `drawBossShip`, and `drawClouds` are the primitives.
  - **Loop**: `loop(now)` runs on `requestAnimationFrame`, computes `dt`, and calls `update` then `draw`. `trackPerf` adapts a `quality` level (0 full, 1 medium, 2 low) when frame times slip.

### Game rules worth knowing

- **Stages**: a stage ends after `KILLS_PER_STAGE` kills, after which a boss spawns. Enemy variety unlocks by stage through `minStage`.
- **Weapons**: `player.weapon` is `vulcan` or `laser`, with `player.level` 1 to 3. `fireWeapon()` sets the spread and lane count from the level.
- **Power-ups**: `weapon` (upgrades or switches weapon), `bomb` (adds a bomb, up to `maxBombs`), and `life` (extra life, up to `maxLives`).
- **Bombs**: `useBomb()` clears enemy bullets, destroys all on-screen enemies (5 points each), and deals 24 damage to the boss. Starting count depends on difficulty.
- **Score multiplier**: kills within a short window build `chain`, which raises `multiplier`. The combo popup reflects this.
- **Difficulty** changes spawn rate, enemy speed, enemy fire chance, starting lives and bombs, and boss HP.

### localStorage keys

These keys are read and written by the game. Changing them resets players' saved state, so treat renames as breaking changes.

| Key | Purpose |
| --- | --- |
| `skyfighter.bestScore` | Best score |
| `skyfighter.muted` | Mute flag (`'1'` or `'0'`) |
| `skyfighter.difficulty` | `easy`, `normal`, or `hard` |
| `skyfighter.reducedMotion` | Reduced-motion override (`'1'` or `'0'`). If unset, the `prefers-reduced-motion` media query decides. |

Every `localStorage` access should stay wrapped so the game still works when storage is blocked (for example in private windows). Keep that in mind when adding new keys.

## Conventions

- **Match the existing style.** The code uses 2-space indentation, single quotes in JS, `const` by default, arrow functions for callbacks, and short section comments such as `// ---------- Audio ... ----------`. Follow that rather than introducing a new pattern.
- **Keep it single-file.** New features go into `index.html` unless there is a clear reason to split. Do not introduce a module system, build tooling, or external assets.
- **No external network dependencies.** The game is self-contained. The only external reference is an inline SVG favicon data URI. Do not add CDN scripts or fonts without asking.
- **Respect reduced motion.** New screen shake, flashes, or pulses must go through `screenShake`, `flashHit`, `flashBomb`, or check `reducedMotion` directly.
- **Announce for screen readers.** Use the `announce(text)` helper (writes to `#srLive`) for important state changes.
- **Keep the logical canvas at 480×720.** `BASE_W` and `BASE_H` define the play field. `resize()` scales the canvas to the viewport, capped at 1.6×. Coordinates in game logic should stay in these base units.
- **Mobile-first controls.** Any new action needs a touch equivalent and must not rely on hover.

## Git workflow

- Develop on the branch specified by the session. Do not push to `main` directly.
- Write descriptive commit messages in the imperative mood, matching the existing history (for example `Add Sky Fighter: ...`).
- Do not open a pull request unless explicitly asked.
