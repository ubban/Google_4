# CLAUDE.md

Guidance for Claude Code when working in this repository.

## What this repo is

`Google_4` is a small static project. Its only real content is **Sky Fighter**, a Raiden-style vertical shoot-em-up built as a single-page HTML5 canvas game.

- `index.html` — the whole game: markup, CSS (`<style>`, ~lines 10–362), canvas element, and one inline IIFE `<script>` (~lines 422–end).
- `README.md` — placeholder title only.
- `CLAUDE.md` — this file.

There is no build step, package manager, test suite, linter, or bundler. Do not introduce one unless the user asks.

## Running it

Open `index.html` directly in a browser, or serve the directory with any static server (e.g. `python3 -m http.server`) and visit it. Nothing needs to be installed.

To verify a change, load the page in a browser and play it. For automated checks, Playwright is available in the cloud environment (`require('playwright')`, `chromium.launch()` with no options).

## Architecture of `index.html`

The script is wrapped in `(function () { 'use strict'; ... })();`. Everything is module-private; there are no globals to preserve.

Major sections, in the order they appear in the script:

1. **DOM references** — `document.getElementById` lookups for HUD, overlay, and touch controls.
2. **Persistence keys and config** — `STORAGE_*` constants, `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES`, `ENEMY_TYPES`, `BOSSES`.
3. **Settings state** — `bestScore`, `muted`, `difficulty`, `reducedMotion`, all read from `localStorage` at load.
4. **Audio** — `ensureAudio()`, `beep()`, `noiseBurst()`, and the `sfx` table. Audio is created lazily on the first user gesture (`ensureAudio()` is called from input handlers).
5. **HUD and FX helpers** — hearts/bombs rendering, hit flash, combo pops, stage banners, screen shake.
6. **Input** — keyboard (`keydown`/`keyup` on `window`), touch joystick and buttons, mouse steering on the canvas.
7. **Game state and spawning** — `resetGame()`, `spawnEnemy()`, `spawnBoss()`, `maybeSpawnPowerUp()`, `spawnExplosion()`.
8. **Scoring** — `addScore()`, `breakChain()` (kill-chain multiplier), `endGame()` (writes best score).
9. **Weapons and bombs** — `fireWeapon()` (Vulcan/Laser, 3 power levels), `useBomb()`, `applyPowerUp()`.
10. **Boss AI** — `BOSS_PATTERNS` (bullet-pattern functions such as `fireRadial`, `fireSpiralBurst`, `fireHomingSeeded`), `updateBoss()`, `killBoss()`.
11. **Update and draw** — `update(dt)` for simulation, `draw()` for rendering, `loop(now)` as the `requestAnimationFrame` driver. `togglePause()` controls pause.
12. **Start** — `startBtn` click handler, then `requestAnimationFrame(loop)`.

Game loop rules:
- Simulation is driven by `dt` passed into `update()`. Keep new gameplay logic frame-rate independent by scaling by `dt`.
- Rendering happens only in `draw()`. Do not mutate game state from draw code.
- Canvas is a fixed logical size `BASE_W × BASE_H` (480 × 720), scaled by `resize()`. Coordinates in game code use the logical size, not CSS pixels.

## Tuning and content

Most balance changes are data edits in the config tables; prefer those over changing logic.

- `DIFFICULTIES` — per-difficulty `spawnBase`, `spawnMin`, `enemySpeed`, `enemyShootChance`, `lives`, `bombs`, `bossHpMul`.
- `ENEMY_TYPES` — `grunt`, `interceptor` (weaves), `tank` (`minStage: 2`). Fields include `hp`, `speedMul`, `score`, and `minStage`.
- `BOSSES` — six bosses cycled by stage, each with colors and `hpMul`. Boss attack behavior lives in `BOSS_PATTERNS`, not in this table.
- `THEMES` — six sky palettes cycled per stage.
- `KILLS_PER_STAGE` — kills needed to advance a stage.

When adding a boss, enemy, or pattern, add the data entry and reuse existing draw/fire helpers where possible.

## Persistence

All persistence goes through `localStorage` with these keys (keep the names stable so saved data survives):

| Key | Meaning | Values |
| --- | --- | --- |
| `skyfighter.bestScore` | Best score | number |
| `skyfighter.muted` | Audio muted | `'1'` / `'0'` |
| `skyfighter.difficulty` | Selected difficulty | `easy` / `normal` / `hard` |
| `skyfighter.reducedMotion` | Reduced motion toggle | `'1'` / `'0'` (absent = follow OS preference) |

Reads and writes to `localStorage` can throw in private windows or with blocked site data. If you add persistence, wrap access in `try/catch` and keep the game working without it. The current code does not guard these calls; if you touch them, consider adding guards.

## Controls (keep the hint text in sync)

The on-screen hint is `#controlsHint`. If you change controls, update it.

- Keyboard: arrows / WASD move, Space fires, X or Shift bombs, P or Esc pauses.
- Mouse: move to steer, click to fire.
- Touch: drag left zone for the joystick, FIRE and BOMB buttons on the right.

Note: `Space` default is prevented on keydown to stop page scroll; keep that when changing input handling.

## Conventions

- **Single file.** Keep HTML, CSS, and JS in `index.html` unless the user asks to split it. Do not add external scripts, fonts, or CDN links without asking.
- **No dependencies.** Plain ES2015+ browser JavaScript. Match the existing style: 2-space indent, `const`/`let`, arrow functions for callbacks, `'use strict'` inside the IIFE.
- **Naming.** `UPPER_SNAKE` for config constants and storage keys; camelCase for functions and state. Enemy and boss type keys are lowercase strings.
- **Accessibility.** Respect `reducedMotion` for screen shake, flashes, and similar effects. Keep the `srLive` live region used by `announce()` for important game events.
- **Performance.** Avoid per-frame allocations in hot paths (`update`, `draw`) where practical; the game already tracks frame cost via `trackPerf()`.

## Git workflow

- Develop on the branch specified by the session (currently `claude/gifted-tesla-3b3ls4`). Do not push to other branches without explicit permission.
- Commit messages in this repo are short imperative summaries followed by bullet detail, as in the existing `Add Sky Fighter` commit.
- Do not open a pull request unless the user asks for one.

## Working agreement for Claude

- Make the smallest change that fixes the problem; this is a single file, so diffs should stay readable.
- After any gameplay change, load `index.html` in a browser (or Playwright) and confirm the affected screen still starts, plays, and ends without console errors.
- When changing balance, state the old and new values in the summary so the user can review the tuning.
- Ask before adding a build system, framework, test runner, or new top-level directories.
