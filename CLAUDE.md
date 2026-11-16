# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a small, dependency-free browser game called **Sky Fighter**: a Raiden-style vertical shoot-'em-up. The entire game lives in a single self-contained file, `index.html` (inline `<style>` and one inline `<script>`, drawn on a `<canvas>`). `README.md` is a placeholder and has no project documentation.

There is no build step, package manager, test suite, or linter. Nothing needs to be installed.

## Repository layout

| Path         | Purpose                                                              |
|--------------|----------------------------------------------------------------------|
| `index.html` | The whole game: head and CSS (~lines 1–359), body markup (~360–421), inline JS (~422–1667) |
| `README.md`  | Placeholder title only                                               |
| `CLAUDE.md`  | This file                                                            |

## Running the game

Open `index.html` directly in a browser, or serve the directory with any static server, for example:

```bash
python3 -m http.server 8000   # then visit http://localhost:8000
```

Verify changes by playing them in a browser (desktop and, where relevant, touch emulation). Type-checking and tests do not exist for this project.

## Architecture of `index.html`

- **HTML/UI**: a `#container` holds the `<canvas id="gameCanvas">`, a HUD (`#hud`: score, best, multiplier, weapon, hearts, bombs), a boss health bar (`#bossBarWrap`), a top-right control bar (`#topBar`: mute, reduced-motion, pause), touch controls (`#touchControls`: joystick zone, FIRE, BOMB), and an `#overlay` start/game-over card with difficulty buttons (`.diffBtn`) and `#startBtn`.
- **Constants** (top of the script):
  - `STORAGE_*` keys (`skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion`) used with `localStorage`.
  - `DIFFICULTIES` (`easy` / `normal` / `hard`): spawn rates, enemy speed and fire chance, starting `lives`, starting `bombs`, and `bossHpMul`.
  - `KILLS_PER_STAGE = 18`; `THEMES` (six sky palettes cycled per stage); `ENEMY_TYPES` (`grunt`, `interceptor`, `tank`, each with `minStage`); `BOSSES` (six bosses, names and palettes, with `hpMul`); `BOSS_PATTERNS` (bullet-pattern functions).
  - `BASE_W = 480`, `BASE_H = 720`: the logical play-field size that `resize()` scales to the viewport.
- **Main loop**: `update(dt)` advances game state, `draw()` renders it, and `loop(now)` drives both with `requestAnimationFrame`. Game state is reset by `resetGame()`.
- **Systems**: spawning (`spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`), weapons (`fireWeapon`: `vulcan` and `laser`), power-ups (`applyPowerUp`: `life`, `bomb`, `weapon`), bombs (`useBomb`), scoring with combo chains (`addScore`, `breakChain`), boss attacks (`updateBoss`, `killBoss`), damage and game end (`loseLife`, `endGame`), pause (`togglePause`).
- **Audio**: synthesized with the Web Audio API (`ensureAudio`, `beep`, `noiseBurst`). `ensureAudio()` is called from key, mouse, and touch handlers so audio starts only after a user gesture.
- **Input**: keyboard (`keys[e.code]`), mouse (`canvasPointFromEvent`, `mousedown`/`mousemove` on the canvas), and touch (`joystickStart`/`joystickMove`/`joystickEnd`, FIRE and BOMB buttons).
- **Accessibility**: `#srLive` (`aria-live`) announcements via `announce()`; a reduced-motion toggle that disables screen shake, flashes, and pulses; `prefers-reduced-motion` is respected as the default.

## Controls (for reference when testing)

- Keyboard: arrows or WASD to move, Space to fire, X or Shift to bomb, P or Esc to pause.
- Mouse: move to steer, click to fire.
- Touch: drag the left zone to fly, tap FIRE, tap BOMB.

## Conventions

- Keep the game as a single self-contained file unless the user asks for a restructure. Do not add a bundler, framework, or dependencies without a clear reason.
- Match the existing style: 2-space indentation, `const`/`let`, camelCase identifiers, UPPER_SNAKE_CASE for tunable constants, and short comments only where they add information.
- Put tunable balance values (speeds, HP, spawn rates, scores) in the existing constant tables rather than hard-coding them in logic.
- Preserve the layout on phone-sized viewports and keep touch controls working when changing the HUD or overlay.
- Keep `localStorage` access in the existing `STORAGE_*` keys; if you add a new persisted setting, follow the same naming pattern (`skyfighter.<name>`).
- Anything that changes how the game looks or plays should be checked in a browser before reporting it as done.

## Git workflow

- Develop on the branch designated for this work (currently `claude/gifted-tesla-b8r1mu`). Do not push to other branches without explicit permission.
- Do not create pull requests unless explicitly asked.
- Commit messages should be short, imperative, and descriptive of the gameplay or file change.
