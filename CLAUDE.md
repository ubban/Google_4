# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

**Sky Fighter** is a single-file, browser-based, Raiden-style vertical shoot-'em-up. The entire game (HTML, CSS, and JavaScript) lives in `index.html`. There is no build step, package manager, bundler, or test suite.

Repository layout:

- `index.html` — the whole game: markup (~lines 1–420), inline `<style>` (lines 10–362), and one IIFE-wrapped `<script>` (starts line 422).
- `README.md` — minimal placeholder.
- `CLAUDE.md` — this file.

## Running and testing

- There is nothing to install or compile. Open `index.html` directly in a browser, or serve the folder with any static server (e.g. `python3 -m http.server`).
- There are no automated tests or linters configured. Verify changes by playing the game in a browser, including the touch controls if the change affects input or layout.
- Syntax-check the inline script after edits if you want a quick sanity pass, e.g. extract the `<script>` block and run `node --check` on it.

## Architecture (inside `index.html`)

The script is one IIFE. Major sections, roughly in source order:

1. **DOM references and constants** — element lookups, `STORAGE_*` localStorage keys (`skyfighter.bestScore`, `skyfighter.difficulty`, mute and reduced-motion flags), and the `DIFFICULTIES` table.
2. **Audio** — `ensureAudio()`, `beep()`, `noiseBurst()`, and the `sfx` object. Audio is created lazily on first user gesture.
3. **UI helpers** — `announce()` (screen-reader live text), HUD rendering (`renderHearts`, `renderBombs`), flash/vignette effects, combo pop, stage banner, and `screenShake()`.
4. **Settings** — `setDifficulty()`, mute and reduced-motion toggles. Settings persist to localStorage.
5. **Input** — keyboard (`keys[e.code]`), mouse (move to steer, click to fire), touch joystick on the left zone, and FIRE and BOMB buttons. There is no device-motion/accelerometer input.
6. **Game state and entities** — `resetGame()`, enemy and boss spawning (`spawnEnemy`, `spawnBoss`), power-ups (`maybeSpawnPowerUp`, `applyPowerUp`), scoring and chain combos (`addScore`, `breakChain`), bombs (`useBomb`), and weapons (`fireWeapon`; `vulcan` and `laser`).
7. **Boss patterns** — `updateBoss`, `killBoss`, and the bullet-pattern helpers (`fireRadial`, `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`, `fireCross`, `fireHomingSeeded`, `fireDoubleRing`).
8. **Main loop** — `loop(now)` is driven by `requestAnimationFrame`, computes `dt` (clamped to 50 ms), then calls `update(dt)` and `draw()` only when running and not paused. `draw()` uses helpers `drawPlane`, `drawBossShip`, `drawClouds`. `trackPerf()` monitors frame time.
9. **Lifecycle** — `loseLife()`, `endGame()`, `togglePause()`, and `resize()` (canvas fit for mobile and desktop).

### Gameplay rules worth knowing

- Difficulty (`easy` / `normal` / `hard`) sets spawn timing, enemy speed, enemy fire chance, starting lives and bombs, and a boss HP multiplier. Changing a number there changes balance across the whole game.
- Each stage ends after `KILLS_PER_STAGE` (18) kills, then a boss fight starts. Spawn rate ramps down over time (`difficultyRamp` in `update`).
- Power-ups: `life`, `bomb` (capped at `maxBombs`), and `weapon` (toggles vulcan/laser).
- Best score is stored in localStorage and shown in the HUD.

## Conventions

- Keep everything in `index.html` unless there is a clear reason to split it. Do not introduce a build system, framework, or dependencies without asking.
- Match the existing style: 2-space indentation, `const`/`let`, small named functions, and `camelCase` identifiers. Constants are `UPPER_SNAKE_CASE`.
- Game-balance numbers are kept inline or in the `DIFFICULTIES` table. Prefer adjusting those over adding new magic numbers in logic.
- Respect the `reducedMotion` setting when adding screen shake, flashes, or pulsing effects (route them through `screenShake()` / existing flash helpers).
- Keep touch and mouse input working when changing controls. The page sets `touch-action: none` and `user-select: none` on purpose.
- Keep the UI readable at phone widths; HUD and button sizes use `clamp()` / `vw` units.

## Git workflow

- Develop on the branch assigned for the session (currently `claude/gifted-tesla-0c4dne`). Do not push to other branches without explicit permission.
- Commit with clear, descriptive messages and push with `git push -u origin <branch>`.
- Do not open a pull request unless explicitly asked.
