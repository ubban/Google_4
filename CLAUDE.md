# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up that runs in the browser. The entire game is a single self-contained file, `index.html` (~1,670 lines: inline `<style>`, markup, and one inline `<script>` IIFE). There is no build step, no bundler, no package manager, no external dependencies, and no CDN calls.

- `README.md` — project name only.
- `index.html` — the whole game.

Persistent state lives in `localStorage` under the `skyfighter.*` keys (best score, mute, difficulty, reduced motion).

## Running and verifying

- Open `index.html` directly in a browser, or serve the directory with any static server (e.g. `python3 -m http.server`).
- There is no test suite, linter, or CI configured. Verify changes by playing the game in a browser, ideally at both desktop and phone-sized viewports, since touch controls and the responsive canvas are part of the game.
- Do not add tooling (package.json, bundlers, frameworks) unless the user asks for it. Keeping the game as one file that opens with no install step is an intentional constraint.

## Code layout inside `index.html`

The script is one IIFE. Rough order of sections, by function name or constant:

| Area | Key identifiers |
|---|---|
| Storage keys and settings | `STORAGE_*`, `bestScore`, `muted`, `difficulty`, `reducedMotion` |
| Quality / perf scaling | `quality`, `trackPerf(dt)` |
| Tuning data | `DIFFICULTIES`, `KILLS_PER_STAGE` (18), `THEMES` (6 stage themes), `ENEMY_TYPES`, `BOSSES` (6 bosses) |
| Audio | `ensureAudio`, `beep`, `noiseBurst`, `sfx` (Web Audio, created lazily on first user gesture) |
| HUD | `renderHearts`, `renderBombs`, `flashHit`, `showComboPop`, `showStageBanner`, `screenShake` |
| Layout and input | `resize` (logical canvas is `BASE_W` × `BASE_H` = 480×720), joystick and mouse handlers, keyboard handlers |
| Game state | `resetGame`, `player`, `bullets`, `enemyBullets`, `powerUps`, `score`, `lives`, `bombCount`, `stage`, `chain`, `multiplier` |
| Spawning | `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp` |
| Combat | `fireWeapon`, `useBomb`, `addScore`, `breakChain`, `applyPowerUp`, `loseLife`, `endGame` |
| Bosses | `BOSS_PATTERNS` (one array of attack functions per boss), `fireRadial`, `fireAimedSpread`, `fireCross`, `fireHomingSeeded`, `updateBoss`, `killBoss` |
| Rendering | `drawPlane`, `drawBossShip`, `drawClouds`, `draw` |
| Loop | `update(dt)`, `loop(now)` driven by `requestAnimationFrame` |

Game flow: the start button calls `resetGame()`. Killing `KILLS_PER_STAGE` enemies sets `bossWarned` and spawns the stage boss after a 1.2 s delay. Defeating a boss advances the stage and theme. Losing all lives calls `endGame()`, which updates the best score.

## Controls (for testing)

- Move: arrow keys / WASD (`keys[e.code]`), or drag the on-screen joystick, or mouse-move over the canvas.
- Fire: Space (auto-fire while held), fire button on touch, mouse-down on the canvas.
- Bomb: `X` or Shift; bomb button.
- Pause: `P` or Escape.

## Conventions

- Keep everything in `index.html` unless the user asks to split it. Match the existing style: 2-space indent, `const`/`let`, single quotes in JS, and plain DOM APIs.
- Game balance lives in the data tables (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `BOSS_PATTERNS`, `THEMES`). Prefer editing those tables over hard-coding numbers in logic.
- Anything that adds per-frame allocation or DOM work inside `update`/`draw` should be checked against the existing `quality` scaling and the mobile target.
- Respect the reduced-motion setting when adding screen shake, flashes, or other motion effects.
- Keep touch and keyboard input paths in parity when adding controls.
- Don't introduce external network requests, analytics, or new third-party assets.

## Git workflow

- Default branch: `main`. Feature work for this environment goes on the branch named in the session instructions.
- Write descriptive commit messages that explain the gameplay or code change.
- Do not open pull requests unless explicitly asked.
