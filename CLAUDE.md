# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo is **Sky Fighter**, a Raiden-style vertical shoot-em-up that runs in the browser. The entire game is a single self-contained file, `index.html` (inline CSS, inline JavaScript, one `<canvas>`). There is no build step, no package manager, no bundler, and no test suite.

Other files:

- `README.md` - placeholder title only.
- `index.html` - the whole game.

## Running the game

Open `index.html` directly in a browser, or serve the directory with any static server, for example:

```bash
python3 -m http.server 8000   # then visit http://localhost:8000/
```

No install step is needed.

## Code layout (`index.html`)

- **Lines ~10-359**: inline `<style>`. Covers the HUD, overlays, touch controls, difficulty buttons, boss bar, and `reduced-motion` styling.
- **Lines ~361-421**: markup. Canvas `#gameCanvas`, HUD elements, overlay, touch controls (`#touchControls`, `#moveZone`, `#fireBtn`, `#bombBtn`), and an `aria-live` region `#srLive`.
- **Lines ~422-1667**: one IIFE holding all game code, in `'use strict'` mode. Nothing is exported to `window`.

Inside the IIFE, the code is grouped by section comments (`// ---------- ... ----------`):

| Area | Key identifiers |
| --- | --- |
| Config tables | `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES`, `ENEMY_TYPES`, `BOSSES`, `BOSS_PATTERNS` |
| Persistence | `STORAGE_*` keys, `bestScore`, `muted`, `difficulty`, `reducedMotion` |
| Adaptive quality | `trackPerf`, `quality` (0 full, 1 medium, 2 low) |
| Audio | `ensureAudio`, `beep`, `noiseBurst`, `sfx` |
| Canvas sizing | `BASE_W = 480`, `BASE_H = 720`, `resize` |
| Input | `keys`, joystick (`joystickStart/Move/End`), mouse and touch handlers, `canvasPointFromEvent` |
| Game state | `resetGame`, `update(dt)`, `applyPowerUp`, `useBomb`, `fireWeapon` |
| Enemies and bosses | `spawnEnemy`, `spawnBoss`, `updateBoss`, `killBoss`, `fire*` pattern helpers |
| Rendering | `draw`, `drawPlane`, `drawBossShip`, `drawClouds`, `spawnExplosion` |
| Lifecycle | `loseLife`, `endGame`, `togglePause`, `loop(now)` (requestAnimationFrame driver) |

## Game rules to keep in mind

- **Difficulty** (`DIFFICULTIES`): `easy`, `normal`, `hard` set spawn timing, enemy speed, enemy shot chance, starting lives, starting bombs, and boss HP multiplier.
- **Stages**: advancing requires `KILLS_PER_STAGE` (18) kills; that triggers a boss. Each boss has its own pattern set and HP multiplier. Stage number selects the theme from `THEMES` (cycled).
- **Enemies**: `grunt` (1 HP), `interceptor` (fast, weaving), `tank` (4 HP, appears from stage 2).
- **Power-ups** (`maybeSpawnPowerUp`): `life`, `bomb`, and `weapon`. Weapon upgrades go level 1 to 3, then switch between `vulcan` and `laser`.
- **Scoring**: kills add score through `addScore`, with a combo multiplier that `breakChain` resets.
- **Controls**: arrows or WASD to move, Space to fire (hold), `X` / Shift for bomb, `P` / Escape to pause. Mouse steers by moving over the canvas and fires on click. Touch uses a drag zone to fly and the on-screen FIRE and BOMB buttons.
- **Persisted in `localStorage`** under the `skyfighter.*` prefix: best score, mute, difficulty, reduced motion. Every access must stay wrapped so the game still works when storage is unavailable.

## Conventions

- Keep everything in `index.html` unless the user asks for a split. Do not introduce a build tool, framework, or npm dependency without an explicit request.
- Match the existing style: 2-space indentation, single quotes in JS, `const` by default, `camelCase` identifiers, `UPPER_SNAKE` for constants, section banner comments in the form `// ---------- Name ----------`.
- Game logic runs off a fixed base canvas (480x720) that is scaled for display. Coordinates and sizes are in base-canvas pixels.
- Keep `dt` handling intact: `loop` clamps `dt` to 0.05 s, and all movement and timers should scale with `dt`.
- Respect `reducedMotion`: screen shake, flashes, and other heavy effects should be gated on it.
- Accessibility: announce important state changes through `announce()` so they reach the `aria-live` region.

## Verifying changes

There are no automated tests or linters. To verify a change:

1. Open `index.html` in a browser (or serve it locally) and play through a few stages.
2. Check the console for errors (`F12`) while exercising the changed code path.
3. Test both keyboard and touch input if the change touches controls, and test difficulty switching and pause if it touches game state.
4. If you can, run a quick syntax check on the inline script, for example by extracting it and running `node --check`.

## Git workflow

- Develop on the branch the task specifies, commit with descriptive messages, and push with `git push -u origin <branch>`.
- Do not open pull requests unless explicitly asked.
