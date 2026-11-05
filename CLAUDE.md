# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up that runs in the browser. The entire game lives in one file, `index.html` (about 1,670 lines): HTML markup, inline CSS (`<style>`, lines ~10–421), and inline JavaScript (`<script>`, lines ~422–1667). There is no framework, bundler, `package.json`, build step, or test suite.

Other files:

- `README.md`: a one-line placeholder (`# Google_4`).
- `CLAUDE.md`: this file.

## Running the game

Open `index.html` directly in a browser, or serve the directory with any static server, for example:

```bash
python3 -m http.server 8000   # then visit http://localhost:8000
```

There is nothing to install and nothing to compile.

## Verifying changes

No automated tests or linters are configured. To verify a change:

1. Load `index.html` in a browser and check the browser console for errors.
2. Exercise the affected path: start a game, fire, use a bomb, collect a power-up, reach a boss, pause/resume, and reach game over.
3. Check both desktop (keyboard/mouse) and touch layouts when touching HUD, overlay, or control code.
4. Syntax-check the inline script if needed, for example by extracting the `<script>` block into a temporary file and running `node --check` on it.

## Code layout inside `index.html`

The inline script is one IIFE, `(function () { ... })();`, that wraps all state and logic.

| Area | Where to look | Notes |
| --- | --- | --- |
| DOM references | top of `<script>` (~L426–461) | `getElementById` lookups. Keep IDs in the markup and here in sync. |
| Storage keys | `STORAGE_*` constants (~L463–466) | `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion`. |
| Difficulty table | `DIFFICULTIES` (~L485) | Spawn rate, enemy speed, enemy fire chance, starting lives and bombs, boss HP multiplier. |
| Stage data | `KILLS_PER_STAGE`, `THEMES`, `ENEMY_TYPES`, `BOSSES` (~L491–516) | Themes cycle through background palettes. Enemies unlock by `minStage`. Bosses have their own hull colors and HP multiplier. |
| Audio | `ensureAudio`, `beep`, `noiseBurst`, `sfx` (~L543–678) | Web Audio API, synthesized at runtime. No audio files. |
| Responsive scaling | `BASE_W = 480`, `BASE_H = 720`, `resize()` (~L753) | The canvas uses a fixed logical resolution and is scaled by CSS, capped at 1.6x. |
| Input | keyboard (~L768), joystick and touch (~L781–834), mouse (~L838–861) | See the controls table below. |
| Game state and reset | `resetGame()` (~L864–910) | Single place where per-run state is initialized. |
| Drawing helpers | `drawPlane`, `drawBossShip`, `drawClouds` (~L913–1020, ~L1507) | Canvas 2D primitives. |
| Spawning | `spawnEnemy`, `spawnBoss` (~L1022–1060) | |
| Update loop | `update(dt)` (~L1148) | Movement, firing, collisions, power-up pickup, stage progression. |
| Power-ups | `applyPowerUp` (~L1300) | Types: `life`, `bomb`, `weapon`. |
| Boss AI | `updateBoss(dt)` (~L1411) | Telegraphed attacks. |
| Render | `draw()` (~L1520) | Everything drawn each frame. |
| Frame loop | `loop(now)` (~L1640) | `requestAnimationFrame`; `dt` is clamped to 50 ms. |
| Start | `startBtn` handler (~L1655) | Calls `resetGame()` and hides the overlay. |

## Game rules to keep consistent

- **Stages:** a stage ends after `KILLS_PER_STAGE` (18) kills. The boss warning fires and the boss spawns about 1.2 s later (~L1204). Defeating the boss clears it and increments `stage` (~L1448). The theme changes with the stage.
- **Weapons:** two types, `vulcan` and `laser`. Fire delay is 0.15 s for vulcan and 0.10 s for laser. Each `weapon` power-up raises `player.level` from 1 to 3; once at level 3, the next pickup switches the weapon type and resets the level to 1 (~L1311).
- **Bombs:** `useBomb()` (~L1103). Bomb count starts at the difficulty value and is capped by `maxBombs`.
- **Lives:** the player starts with `lives` from `DIFFICULTIES`. Invulnerability frames follow a hit (`player.invuln`).
- **Score and multiplier:** kills feed a chain counter and a multiplier shown in the HUD. Chain expires when `chainTimer` runs out.
- **Best score:** saved to `localStorage` on game over when beaten (~L1477). Storage access is not guarded, so keep future storage calls in try/catch if you add more.
- **Game over:** `endGame()` (~L1469) sets `gameOver`, updates the overlay stats, and shows the start screen.
- **Pause:** `P` or `Esc` toggles `togglePause()` (~L1491). The start button also resumes a paused game.

## Controls

| Input | Action |
| --- | --- |
| Mouse move | Steer |
| Mouse click (canvas) | Fire |
| Arrows / WASD | Move |
| Space | Fire |
| X or Shift | Bomb |
| P or Esc | Pause |
| Touch drag on left zone | Virtual joystick |
| FIRE button / BOMB button | Fire / bomb |

## Conventions

- **Single file:** keep CSS and JS inline in `index.html` unless the user asks to split it. Do not add a build system or dependencies without asking.
- **Vanilla JS only:** no libraries, no ES module imports, and no transpilation. Target modern evergreen browsers.
- **Style:** match the existing 2-space indentation, single quotes in JS, `const`/`let` (no `var`), and the arrow-function and template-literal style already used.
- **State:** mutable game state lives in the IIFE's `let` variables. Reset everything per run in `resetGame()`, not piecemeal in other functions.
- **Units:** positions and sizes are in logical canvas pixels (480×720). Speeds are in pixels per second and are multiplied by `dt`.
- **Accessibility:** respect `prefers-reduced-motion` (the `reducedMotion` flag and motion toggle). Keep the `srLive` live region announcements (`announce()`) working for important state changes.
- **Mobile:** the page disables zoom and scrolling (`touch-action: none`). Preserve touch targets when changing the HUD or control layout.

## Making changes

- Read the relevant section before editing. Most logic is in `update`, `draw`, and the helpers they call.
- Keep changes minimal and local. Do not reformat unrelated code.
- When adding a new enemy, boss, theme, or power-up, add it to the existing data table and handle it in the matching update, draw, and apply functions.
- After editing, run the verification steps above before committing.

## Git

- Default development branch for this workspace: `claude/gifted-tesla-hvwqgh`. Do not push to other branches without explicit permission.
- Write clear, descriptive commit messages.
- Do not open a pull request unless the user asks for one.
