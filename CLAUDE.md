# CLAUDE.md

Guidance for working in this repository.

## Project overview

Google_4 is a small static web project containing **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs in a browser canvas.

- `index.html` — the entire game: markup, CSS, and JavaScript in one file (~1,670 lines).
- `README.md` — minimal placeholder.

There is no build step, package manager, bundler, test suite, or linter. Do not add one unless asked.

## Running the game

Open `index.html` directly in a browser, or serve the folder with any static server (for example `python3 -m http.server`). No dependencies are fetched at runtime.

## Code layout inside `index.html`

- **Lines ~1–360:** `<head>` metadata (inline SVG favicon) and the `<style>` block. Styles use plain CSS; the HUD, overlay, touch controls, and boss bar are all styled here.
- **Lines ~361–421:** DOM markup. Key containers: `#container`, `#gameCanvas`, `#hud`, `#bossBarWrap`, `#stageBanner`, `#topBar` (mute / reduced-motion / pause buttons), `#touchControls`, and `#overlay` (start / game-over card with difficulty buttons).
- **Lines ~422–1668:** a single IIFE holding all game logic. Sections, roughly in order:
  - DOM references and `STORAGE_*` keys
  - Performance tracking (`trackPerf`, `quality` 0/1/2 — lowers particle counts and glow when frames run slow)
  - Tuning tables: `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES`, `ENEMY_TYPES`, `BOSSES`
  - Persistence and settings (best score, mute, difficulty, reduced motion)
  - Audio: Web Audio helpers `beep`, `noiseBurst`, and the `sfx` table. Audio is created lazily by `ensureAudio()`, which must run from a user gesture.
  - HUD helpers (`renderHearts`, `renderBombs`, `showComboPop`, `showStageBanner`, `screenShake`)
  - Input: keyboard (`keys` map), virtual joystick and buttons for touch, and mouse steering (`mouseTarget`)
  - Game state and `resetGame()`
  - Rendering: `drawPlane`, `drawBossShip`, `draw()`
  - Spawning: `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`
  - Combat: `fireWeapon`, `useBomb`, `applyPowerUp`, boss attack patterns in `BOSS_PATTERNS` (indexed by boss design), `updateBoss`, `killBoss`, `loseLife`, `endGame`, `togglePause`
  - `update(dt)` (per-frame simulation), `loop(now)` (requestAnimationFrame driver, `dt` clamped to 50 ms)

## Game mechanics reference

- **Stages:** reaching `KILLS_PER_STAGE` (18) kills triggers a boss, which spawns 1.2 s later. Defeating the boss (`killBoss()`) increments `stage` and resets the kill count. Each stage picks a background theme with `(stage - 1) % THEMES.length` and a boss with `(stage - 1) % BOSSES.length`.
- **Enemies:** `grunt` (stage 1+), `interceptor` (stage 1+, weaves), `tank` (stage 2+, 4 HP).
- **Difficulty** (`easy` / `normal` / `hard`) sets spawn rate, enemy speed, enemy fire chance, starting lives, starting bombs, and a boss HP multiplier.
- **Weapons:** power-ups upgrade the weapon level (spread shots at level 2 and 3, laser variant). Bombs are capped by `maxBombs` from the difficulty.
- **Scoring:** points are multiplied by a chain-based `multiplier`; `breakChain()` resets it.

## Persistence

`localStorage` keys, all prefixed `skyfighter.`:

| Key | Values |
| --- | --- |
| `skyfighter.bestScore` | number |
| `skyfighter.muted` | `'1'` / `'0'` |
| `skyfighter.difficulty` | `easy` / `normal` / `hard` |
| `skyfighter.reducedMotion` | `'1'` / `'0'` (if absent, follows `prefers-reduced-motion`) |

Reads and writes are not wrapped in try/catch. If you touch them, consider that storage can be blocked (private windows, disabled site data).

## Controls

- **Desktop:** mouse to steer and click to fire; arrows or WASD to move; Space to fire; X or Shift for bomb; P or Esc to pause.
- **Touch:** drag in the left zone to fly (virtual joystick); tap FIRE; tap BOMB.

## Conventions

- Keep everything in `index.html` unless the user asks for a split. Match the existing style: 2-space indent, single quotes, `const`/`let`, short helper functions, and terse comments only where the logic is not obvious.
- Tuning values live in the constant tables near the top of the script (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `BOSS_PATTERNS`, `THEMES`). Prefer changing those over hard-coding numbers in logic.
- The canvas uses a fixed logical size (`BASE_W` 480 × `BASE_H` 720) scaled by `resize()`. Keep coordinates in logical units.
- Respect the reduced-motion setting when adding screen shake, flashes, or pulsing effects (`reducedMotion` gate, `screenShake`, `flashHit`).
- Any new sound should go through `beep`, `noiseBurst`, or `sfx`, and must be safe to call before `ensureAudio()` has run.
- Changes that affect rendering cost should respect `quality` so low-end devices stay playable.

## Verification

No automated tests exist. To verify a change, load `index.html` in a browser, play through at least one stage boss on each difficulty, and check the browser console for errors. For touch changes, also check a touch-emulated viewport.

## Git

- Develop on the branch designated for the task (currently `claude/gifted-tesla-d7a08f`). Do not push to `main` directly.
- Write descriptive commit messages in the imperative mood.
