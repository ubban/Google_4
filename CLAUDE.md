# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a small static web project. Its only real content is **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs in a browser canvas.

| File | Purpose |
| --- | --- |
| `index.html` | The entire game: markup, CSS, and JavaScript in one file (~1,670 lines). |
| `README.md` | Placeholder. Its content is just the project name and a timestamp. |

There is no `package.json`, build step, bundler, linter, or test suite. Nothing needs to be installed.

## Running the game

Open `index.html` directly in a browser, or serve the folder with any static server, for example:

```bash
python3 -m http.server 8000   # then visit http://localhost:8000/index.html
```

Verify changes by playing them in a browser. There are no automated checks to run.

## Architecture of `index.html`

The file has three sections, in order:

1. **`<style>`** (about lines 10–360): layout, HUD, overlays, and touch controls. Layout must work on both desktop and mobile.
2. **`<body>`** (about lines 361–421): the `#gameCanvas` element, HUD elements, overlays (start, pause, game over), and touch controls (`moveZone`, fire button, bomb button).
3. **`<script>`** (about lines 422–1668): one IIFE, `(function () { ... })();`. All game state and functions are private to it. Nothing is exposed on `window`, so keep it that way unless there is a reason to change it.

### Main pieces inside the script

- **Configuration constants** near the top: `DIFFICULTIES` (spawn rate, enemy speed, shot chance, lives, bombs, boss HP multiplier), `KILLS_PER_STAGE`, `THEMES` (six sky palettes that cycle by stage), `ENEMY_TYPES` (`grunt`, `interceptor`, `tank`), and `BOSSES` (six bosses, cycled in order).
- **Persistence**: `localStorage` keys prefixed `skyfighter.` (`bestScore`, `muted`, `difficulty`, `reducedMotion`). Read and write them defensively, because storage can be unavailable.
- **Audio**: a lazily created Web Audio context (`ensureAudio`), with helpers `beep`, `noiseBurst`, and a `sfx` object. Audio starts only after a user gesture.
- **Input**: keyboard (`keys` map), touch joystick (`joystickStart/Move/End`), fire and bomb buttons, and mouse on the canvas.
- **Rendering**: `draw()` runs each frame, with helpers `drawPlane`, `drawBossShip`, and `drawClouds`. `resize()` scales a fixed `BASE_W × BASE_H` (480 × 720) logical canvas to the window.
- **Loop**: `loop(now)` calls `update(dt)` then `draw()` via `requestAnimationFrame`. `trackPerf` lowers the `quality` level (full, medium, low) when frames run slow.
- **Game logic**: `update(dt)` handles movement, firing, collisions, enemy and bullet updates, and power-up pickup. `updateBoss(dt)` drives boss attack patterns (`BOSS_PATTERNS`). `applyPowerUp` handles `life`, `bomb`, and `weapon` pickups.
- **State**: module-level `let` variables (`score`, `lives`, `stage`, `chain`, `multiplier`, `player`, `enemies`, `bullets`, and so on). `resetGame()` restores them to starting values.

## Game rules, for reference

- **Controls**: arrow keys or WASD move (`keys` tracks `e.code`). Space fires. `X` or Shift triggers a bomb. `P` or Escape pauses. On touch devices, drag the left zone to move and tap the fire button.
- **Lives and bombs** depend on difficulty. Normal starts with 3 lives and 2 bombs.
- **Stages**: after `KILLS_PER_STAGE` (18) kills, a boss fight starts. Defeating the boss opens the next stage, which uses the next theme and the next boss, both cycling by `(stage - 1) % length`.
- **Scoring**: kills add points scaled by a chain multiplier. The chain breaks after 2.2 seconds with no kills (`chainTimer`).
- **Power-ups** drop from destroyed enemies: `life`, `bomb`, and `weapon`. The weapon power-up upgrades the current weapon to level 3, then switches between `vulcan` and `laser`.

## Conventions

- **Keep it one file.** The project is intentionally a single self-contained HTML file with no dependencies, so do not add a build step, a framework, or external assets unless the user asks for it.
- **Match the existing style.** Use 2-space indentation, `const` by default, arrow functions for callbacks, and short comments only where the logic is not obvious. Name constants in `UPPER_SNAKE_CASE`.
- **Keep game tuning in the constants.** Put numbers such as speeds, spawn rates, HP, and scores in the existing config objects (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`) rather than hard-coding them in the logic.
- **Respect the pause and reduced-motion settings.** New visual effects should check `reducedMotion` and `quality`. New gameplay should work when paused and across both difficulties.
- **Test on mobile width.** Any layout or input change should be checked at phone size and with touch input, not only with a mouse.
- **Keep audio behind a user gesture.** Create the audio context through `ensureAudio()`, and do not play sound before a click or key press.

## Git workflow

- Develop on the branch the session names. Do not push to other branches without permission.
- Write clear, descriptive commit messages in the imperative mood.
- Do not open a pull request unless the user asks for one.
