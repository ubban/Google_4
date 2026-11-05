# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo is **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs in the browser. The whole game lives in one self-contained file, `index.html`: markup, CSS, and a single inline `<script>` that draws to a `<canvas>` and synthesizes its sound effects with the Web Audio API.

Files:

- `index.html` — the entire game (about 1,670 lines).
- `README.md` — minimal project readme.

There is no `package.json`, no build step, no bundler, no test suite, and no linter configuration.

## Running the game

Open `index.html` directly in a browser, or serve the directory with any static server, for example:

```bash
python3 -m http.server 8000   # then visit http://localhost:8000
```

Nothing needs to be installed or compiled.

## Architecture

Everything runs inside one `(function () { ... })();` block (lines ~423–1666). Rough layout of `index.html`:

- **Styles (top of file, `<style>`)** — dark theme via CSS custom properties and `color-scheme: dark`. HUD, overlays, cards, and keyframe animations (`heartBeat`, `flashHit`, `comboPulse`, etc.). Mobile touch controls are styled here too.
- **Markup (`<body>`)** — a `#container` holding the `#gameCanvas`, the HUD, the overlay/menu cards, and the mobile joystick and FIRE/BOMB buttons. The control hint text near line 417 documents the input scheme.
- **Constants (script start)** — `STORAGE_*` localStorage keys (`skyfighter.bestScore`, `.muted`, `.difficulty`, `.reducedMotion`), `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES` (per-stage palettes), `ENEMY_TYPES`, and `BOSSES`.
- **Audio** — `ensureAudio()`, `beep()`, `noiseBurst()`, `panFor()`. Audio is created lazily on the first user gesture, so keep it that way for browser autoplay rules.
- **Input** — keyboard (`keys` map on `keydown`/`keyup`), mouse on the canvas (`mousemove`/`mousedown`), and touch (`joystickStart`/`joystickMove`/`joystickEnd`, fire and bomb buttons). Keyboard and joystick input take priority when active.
- **Game state and loop** — `resetGame()` initializes state; `update(dt)` advances the simulation; `draw()` renders each frame; `loop(now)` drives both with a delta-time step. Pause is handled by `togglePause()`.
- **Entities and combat** — player weapons via `fireWeapon()`, power-ups (`maybeSpawnPowerUp`, `applyPowerUp`), enemies (`spawnEnemy`), bosses (`spawnBoss`, `updateBoss`, `killBoss`), and boss bullet patterns in `BOSS_PATTERNS` (radial, aimed spread, columns, spiral, cross, homing, double ring). Scoring and combos use `addScore()` and `breakChain()`. Bombs use `useBomb()`.
- **Lives and game over** — `loseLife()` and `endGame()`.
- **Accessibility and polish** — `announce()` for screen-reader live regions, a reduced-motion toggle (`updateMotionBtn`), `screenShake`, and a simple `trackPerf` frame-time monitor.

## Controls (keep the hint text in sync)

- Keyboard: Arrows/WASD move, Space fire, X or Shift bomb, P or Esc pause.
- Mouse: move to steer, click to fire.
- Touch: drag left side to fly, tap FIRE, tap BOMB.

If you change a control, update both the code and the `#controlsHint` text.

## Conventions

- Keep the game as a single self-contained HTML file unless the user asks for a restructure. Do not introduce a build step or external dependencies without a clear reason.
- Do not add new external network requests or CDN scripts; the game should run offline from the file alone.
- Preserve the existing style: 2-space indentation, `const`/`let`, and small named functions per concern.
- Game tuning values (speeds, spawn rates, difficulty multipliers, boss patterns) live in the constants and `update*` functions. Prefer changing those over restructuring logic.
- Persist only the four `skyfighter.*` localStorage keys. Storage reads at startup are currently not wrapped in `try/catch`; when touching them, wrap access so the game still runs if storage is blocked (e.g., private mode or disabled site data).
- Any new sound must go through `beep()` or `noiseBurst()` so mute and panning keep working.
- Respect `prefers-reduced-motion` and the in-game reduced-motion toggle when adding screen shake, flashes, or animations.

## Verifying changes

There is no automated test suite. To verify a change:

1. Open `index.html` in a browser (or serve it locally) and check the browser console for errors.
2. Play through at least one stage, a boss fight, bomb use, pause/resume, and a game over.
3. Check touch controls with browser device emulation if the change touches input or layout.
4. Confirm the HUD and overlays still fit at phone width.

## Git

- Development branch for this work: `claude/gifted-tesla-y0yei7`. Push there with `git push -u origin <branch>`.
- Write clear, descriptive commit messages in the imperative mood.
- Do not open a pull request unless the user explicitly asks for one.
